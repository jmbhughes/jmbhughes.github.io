+++
title = 'Making stars'
date = 2024-06-22
draft = false
toc = true
+++

## Why make stars?

## Making the first version

### Imports and packages

```py
import pandas as pd
```

### Getting a catalog

```py
HIPPARCOS_URL = "https://cdsarc.cds.unistra.fr/ftp/cats/I/239/hip_main.dat"

def load_raw_hipparcos_catalog(
    catalog_path: str = HIPPARCOS_URL
    ) -> pd.DataFrame:
    """Download hipparcos catalog from website.

    Parameters
    ----------
    catalog_path : str
        path to the Hipparcos catalog

    Returns
    -------
    pd.DataFrame
        loaded catalog with selected columns
    """
    column_names = (
        "Catalog",
        "HIP",
        "Proxy",
        "RAhms",
        "DEdms",
        "Vmag",
        "VarFlag",
        "r_Vmag",
        "RAdeg",
        "DEdeg",
        "AstroRef",
        "Plx",
        "pmRA",
        "pmDE",
        "e_RAdeg",
        "e_DEdeg",
        "e_Plx",
        "e_pmRA",
        "e_pmDE",
        "DE:RA",
        "Plx:RA",
        "Plx:DE",
        "pmRA:RA",
        "pmRA:DE",
        "pmRA:Plx",
        "pmDE:RA",
        "pmDE:DE",
        "pmDE:Plx",
        "pmDE:pmRA",
        "F1",
        "F2",
        "---",
        "BTmag",
        "e_BTmag",
        "VTmag",
        "e_VTmag",
        "m_BTmag",
        "B-V",
        "e_B-V",
        "r_B-V",
        "V-I",
        "e_V-I",
        "r_V-I",
        "CombMag",
        "Hpmag",
        "e_Hpmag",
        "Hpscat",
        "o_Hpmag",
        "m_Hpmag",
        "Hpmax",
        "HPmin",
        "Period",
        "HvarType",
        "moreVar",
        "morePhoto",
        "CCDM",
        "n_CCDM",
        "Nsys",
        "Ncomp",
        "MultFlag",
        "Source",
        "Qual",
        "m_HIP",
        "theta",
        "rho",
        "e_rho",
        "dHp",
        "e_dHp",
        "Survey",
        "Chart",
        "Notes",
        "HD",
        "BD",
        "CoD",
        "CPD",
        "(V-I)red",
        "SpType",
        "r_SpType",
    )
    df = pd.read_csv(
        catalog_path,
        sep="|",
        names=column_names,
        usecols=["HIP", "Vmag", "RAdeg", "DEdeg", "Plx"],
        na_values=["     ", "       ", "        ", "            "],
    )
    df["distance"] = 1000 / df["Plx"]
    df = df[df["distance"] > 0]
    return df.iloc[np.argsort(df["Vmag"])]
```

### Filtering for relevant stars

```py
def filter_for_visible_stars(
    catalog: pd.DataFrame, 
    dimmest_magnitude: float = 6
    ) -> pd.DataFrame:
    """Filters to only include stars brighter than a given magnitude

    Parameters
    ----------
    catalog : pd.DataFrame
        a catalog data frame

    dimmest_magnitude : float
        the dimmest magnitude to keep

    Returns
    -------
    pd.DataFrame
        a catalog with stars dimmer than the `dimmest_magnitude` removed
    """
    return catalog[catalog["Vmag"] < dimmest_magnitude]
```

```py
def find_catalog_in_image(
    catalog: pd.DataFrame, 
    wcs: WCS, 
    image_shape: (int, int),
    mode: str = "all"
    ) -> np.ndarray:
    """Using the provided WCS converts the RA/DEC catalog into pixel coordinates

    Parameters
    ----------
    catalog : pd.DataFrame
        a catalog dataframe
    wcs : WCS
        the world coordinate system of a given image
    image_shape: (int, int)
        the shape of the image array associated with the WCS, 
        used to only consider stars with coordinates in image
    mode : str
        either "all" or "wcs",
        see
        <https://docs.astropy.org/en/stable/api/astropy.coordinates.SkyCoord.html#astropy.coordinates.SkyCoord.to_pixel>

    Returns
    -------
    np.ndarray
        pixel coordinates of stars in catalog that are present in the image
    """
    try:
        xs, ys = SkyCoord(
            ra=np.array(catalog["RAdeg"]) * u.degree,
            dec=np.array(catalog["DEdeg"]) * u.degree,
            distance=np.array(catalog["distance"]) * u.parsec,
        ).to_pixel(wcs, mode=mode)
    except NoConvergence as e:
        xs, ys = e.best_solution[:, 0], e.best_solution[:, 1]
    bounds_mask = (0 <= xs) * (xs < image_shape[0]) * (0 <= ys) * (ys < image_shape[1])
    reduced_catalog = catalog[bounds_mask].copy()
    reduced_catalog["x_pix"] = xs[bounds_mask]
    reduced_catalog['y_pix'] = ys[bounds_mask]
    return reduced_catalog
```

### Making the actual star image

```py
def simulate_star_image(wcs, 
                        img_shape, 
                        fwhm,
                        wcs_mode: str = 'all',
                        mag_set=0,
                        flux_set=500_000,
                        noise_mean: float | None = 25.0,
                        noise_std: float | None = 5.0,
                        dimmest_magnitude=8):
    sigma = fwhm / 2.355

    catalog = load_hipparcos_catalog()
    filtered_catalog = filter_for_visible_stars(catalog,
                                                dimmest_magnitude=dimmest_magnitude)
    stars = find_catalog_in_image(filtered_catalog,
                                  wcs,
                                  img_shape,
                                  mode=wcs_mode)
    star_mags = stars['Vmag']

    sources = QTable()
    sources['x_mean'] = stars['x_pix']
    sources['y_mean'] = stars['y_pix']
    sources['x_stddev'] = np.ones(len(stars))*sigma
    sources['y_stddev'] = np.ones(len(stars))*sigma
    sources['flux'] = flux_set * np.power(10, -0.4*(star_mags - mag_set))
    sources['theta'] = np.zeros(len(stars))

    fake_image = make_gaussian_sources_image(img_shape, sources)
    if noise_mean is not None and noise_std is not None:  # we only add noise if it's specified
        fake_image += make_noise_image(img_shape, 'gaussian', mean=noise_mean, stddev=noise_std)

    return fake_image, sources
```

```py

m = 1
shape = (1024*m, 1024*m)
w = WCS(naxis=2)
w.wcs.crpix = shape[1] / 2 - 0.5, shape[0] / 2 - 0.5
w.wcs.crval = 0, 0
w.wcs.cdelt = 0.0225, 0.0225
w.wcs.ctype = 'RA---ARC', 'DEC--ARC'

img, stars = simulate_star_image(
    w, 
    shape, 
    fwhm=3, 
    dimmest_magnitude=12, 
    noise_mean=None, 
    noise_std=None)
```

```py
fig, ax = plt.subplots()
ax.imshow(
    img, 
    vmin=0, 
    vmax=50, 
    cmap='Greys_r'
    )
ax.set_axis_off()
fig.savefig(
    "first.png", 
    bbox_inches='tight'
    )
```

![first star image](first.png)

## Applying a non-uniform point-spread function

![zoomed in star iamge](zoom.png)



## Complete script
