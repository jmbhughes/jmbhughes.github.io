+++
title = "News"
template = "info-page.html"
path = "news"

[extra]
mermaid = true
+++

This is an incomplete view of my accomplishments and work history.

{% timeline() %}
[
    {
        "title":"Promoted to Senior Computer Scientist",
        "body":"I was promoted to Senior Computer Scientist at SwRI and given the official title of PUNCH Science Operations Center Manager.",
        "date":"Dec-2023"
    },
    {
        "title":"Published paper on point spread function regularization",
        "body":"Published Coma Off It: Regularizing Variable Point-spread Functions in the Astronomical Journal",
        "date":"Apr-2023"
    },
    {
        "title":"Started at Southwest Research Institute",
        "body":"Began working as a Research Computer Scientist creating the PUNCH data reduction pipeline.",
        "date":"Jun-2021"
    },
    {
        "title":"Published my first paper",
        "body":"Published Real-time solar image classification: Assessing spectral, pixel-based approaches in the Journal of Space Weather and Space Climate",
        "date":"Sep-2019"
    },
    {
        "title":"Graduated college",
        "body":"Graduated from Williams College with B.A. in computer science and astronomy.",
        "date":"Jun-2018"
    }
]
{% end %}
