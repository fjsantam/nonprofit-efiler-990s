# nonprofit-efiler-990s
Demonstration of tools that can be used to identify efiled IRS 990 tax returns and matching them to EINs.

[Link to the project](https://fjsantam.github.io/nonprofit-efiler-990s/Identifying-EINs-with-efiler-data-210310.html)

## About this project

Part of this work was inspired by requests from [Dr. David Suarez](https://evans.uw.edu/profile/david-suarez/) on how to identify whether organizations have efiled annual tax returns and, if they have, the most recent year of available data. Special thanks to [Dr. Jesse Lecy](http://www.lecy.info/) and the [Nonprofit Open Data Collective](https://nonprofit-open-data-collective.github.io/) for hosting data and code, as well as [Dr. David Borenstein](https://www.open990.org/contact/) (please review his overview articles on [what the IRS 990 efile dataset is](https://medium.com/@borenstein/the-irs-990-e-file-dataset-getting-to-the-chocolatey-center-of-data-deliciousness-90f66097a600) and [why the IRS-provided index files are flawed](https://appliednonprofitresearch.com/posts/2020/06/skip-the-irs-990-efile-indices/)).

## Background on the 990 efiler data

The IRS 990 efiler data "liberation" project began in earnest in 2016 and 2017, with various organizations, academics, and individuals coming together in Spring 2017 at an event hosted by the Aspen Institute in Washington, D.C. to work towards creating a "concordance file" that mapped fields on the forms to the various pathways in the XML files produced by the IRS and hosted by Amazon Web Services (general information [here](https://registry.opendata.aws/irs990/)). The IRS [produces files that act as indices](https://docs.opendata.aws/irs-990/readme.html) of all returns electronically filed in a given period. While they can have errors, as noted by Dr. Borenstein, the index files ultimately yield readily accessible data and, more importantly, the web addresses or URLs of the XML files for a significant portion of the efiled 990s.

For additional information on scraping data from the XML files, please refer to the links above.
