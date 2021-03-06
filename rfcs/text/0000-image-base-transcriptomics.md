**Adding DCP support for image based-transcriptomics**
======================================================

**Summary**
-----------

The GG has requested that the DCP prioritize the incorporation of an
image-based transcriptomics assay into the DCP. This RFC describes the
minimum set of deliverables that must be prioritized and completed for
the DCP to be able to (1) store and (2) process image-based single-cell
transcriptomics experiments. The following assumptions are made:

1.  Support for an image-based transcriptomics assay in the early
    > iteration phase should be added to the DCP

2.  The current state of the starfish ecosystem is sufficient to fulfill
    > this need and will be selected by the AWG.

3.  This instantiation will require limited scale (low hundreds of
    > datasets over several years)

**Author(s)**
-------------

\[Ambrose Carr\](mailto:acarr\@chanzuckerberg.com)

\[Josh Moore\](mailto:j.a.moore\@dundee.ac.uk)

**Shepherd**
------------

**Motivation**
--------------

Cellular imaging assays typically either (1) categorize cells based on
the presence/absence of a particular set of markers, or (2) count
structures within cells. These assays exist at variable levels of
magnifications, including the counting of [[individual
atoms]{.underline}](https://www.sciencemag.org/news/2018/07/tour-de-force-researchers-image-entire-fly-brain-minute-detail)
(Electron Microscopy (EM))
[[synapses]{.underline}](https://www.hhmi.org/news/how-to-rapidly-image-entire-brains-at-nanoscale-resolution)
(lattice light sheet), or [[counting molecules within
cells]{.underline}](http://linnarssonlab.org/osmFISH/) (image-based
single-cell transcriptomics and image-based proteomics).

Image-based transcriptomics assays are a good candidate for DCP imaging
support. They are relatively simple imaging assays that transform image
features into a series of detected transcripts, which are aggregated by
cell. Because these assays require thin-sections of tissue, they produce
datasets that are relatively small by imaging standards (1-10s TBs). In
addition, image-based transcriptomics pipelines generate the same cell x
gene expression matrix outputted by scRNA-seq assays with additional
cellular annotations of the location of the cells. Because these data
structures are already present in the DCP, there are fewer components
that need to be generalized than for other imaging assays. Finally,
engineers from
[*[starfish]{.underline}*](https://spacetx-starfish.readthedocs.io/en/latest/)
and [[OME]{.underline}](https://www.openmicroscopy.org/) are working
together to craft data formats, validation tooling, and processing
pipelines that comply with DCP requirements where possible. These
characteristics make this assay class a good choice for prototyping
image support in the DCP as it presents the smallest scale challenge,
the majority of the major deliverables can be provided by DCP-extrinsic
developers, and it requires generalization of the fewest DCP components.
Implementation of support for these assays is likely to generalize to
additional areas of interest to the HCA community such as multiplexed
antibody imaging and thick-section volumetric imaging for both RNA and
protein.

The remainder of this document provides additional details on the
characteristics of image-based transcriptomics assays, why they are the
right assay class to prototype imaging support in the DCP, their current
scientific state, the standardization of imaging file formats, and a
longer-term view of support for single-cell imaging assays. In brief:

-   Support for image-based transcriptomics can be added without need to
    > construct any additional DCP services, and should not require
    > significant generalization of existing ones.

-   Image-based transcriptomics is the right class of assay to initiate
    > imaging support in the DCP because of their similarity to existing
    > DCP sequencing assays and the abundance of labs using
    > single-molecule FISH assays to clarify spatial distributions of
    > cell types identified in RNA-seq experiments.

-   If desired, *all* key validation, specification, and analysis
    > deliverables can be provided by *starfish* and OME engineers,
    > minimizing load on DCP engineering teams.

-   A draft metadata schema has already been built by EBI via
    > collaboration with the SpaceTx consortium, who have delivered
    > image-based transcriptomics datasets which the ingestion service
    > is beginning to test.

-   Based on [[delivery timelines for pre-requisite
    > work]{.underline}](https://docs.google.com/document/d/1RQcj450v0WHGUvSm7YwwFQ7sRFoPAOCjRbo63UzyQd0/edit?ts=5c40d3f1#heading=h.4qbf4q96id16),
    > implementation of support for image-based transcriptomics could
    > begin as early as Q2-Q3 2019.

-   While image-based transcriptomics data can be ingested and stored
    > without a visualization capability, significant user demand can be
    > expected similar to that for count matrices.

### **User Stories**

-   As a submitter, I want to upload directories of files representing
    > an imaging experiment for storage in the DCP.

-   As a user with a keyboard, I want to identify images with a given
    > gene for download.

-   As a user with a keyboard, I want to see spots overlayed on an image
    > for QC.

-   As a user with a keyboard, I want to see cells overlayed on an image
    > for QC.

-   As a user with a keyboard, I want to generate a gene x cell matrix
    > for biological analysis of my data and combination with sequencing
    > assays.

**Scientific \"guardrails\"**
-----------------------------

Image-based transcriptomics assays are still in active development, and
benchmarking and characterization of these data types is in process. As
such we expect to utilize community feedback to iterate on scientific
quality of the pipelines, and will seek governance approval for the
inclusion of data from these pipelines in release only when the
technologies are more mature.

Before large-scale ingress of these complex datasets is begun, the
proposed integration should be reviewed against the HCA Science
governance's understanding of both the communities likely use of imaging
data as well as the long-term costs of storage and transfer. Based on
the intended investment for imaging, additional effort at this initial
stage of the architecture may be called for to increase value or
decrease cost.

**Detailed Design**
-------------------

### Data Specification

A data specification consists of format specifications for any files
(inputs and outputs) used by imaging assays, and HCA metadata that sits
on top of these data and describes information about the samples and how
the data were acquired. Formats and specifications listed below are
being developed by *starfish*, OME, and HCA Metadata teams. Each have
working prototypes supported by active development teams.

#### Input Data Formats

##### SpaceTx Format

To prototype the use of imaging data in the DCP, the *starfish* and OME
teams propose that the DCP use [[SpaceTx
Format]{.underline}](https://spacetx-starfish.readthedocs.io/en/latest/sptx-format/index.html),
a format that groups and indexes a potentially very large number of two
dimensional images, forming 5-dimensional tensors over (time, channel,
z, y, x). These dimensions are fairly consistent across imaging domains
and will likely generalize to other use cases. SpaceTx created this
format because they were unable to find an existing format that
satisfied our cloud compute and data visualization use cases. *starfish*
and OME have since become aware of contemporaneous development of
projects that aim to satisfy similar use cases and are working towards
unification, where possible across scientific disciplines and languages.

To avoid writing a large amount of microscope-specific code at the
outset, *starfish* requests that users carry out basic pre-processing of
their data to fix registration or chromatic errors. More details on the
specific requests made of users can be found
[[here]{.underline}](https://drive.google.com/a/chanzuckerberg.com/open?id=1x516WMMWkEKKQwLrqqBsKmsPEDNjFYugqxm400D2NjY).The
exact specification of the image data format will depend upon the
delivery date that the DCP requests for this prototype.

##### Pipeline Recipes

The parameterization of an imaging pipeline depends on the assay in
question, the microscope used to capture the data, the density of spots
in the tissue, and the tissue being analyzed. For example, if the same
assay is run a second time at twice the magnification, spots would be
expected to have twice the radius, which would need to be communicated
to the spot-finding algorithm.

Human analysts are very good at selecting these parameters, but
processes to fit parameters to all datasets across assays, tissues, and
microscopes have not yet been developed. To solve this problem, starfish
exposes the ability for users to pilot starfish on small subsets of
their data in a local environment. These analyses generate parameter
sets (\"pipeline recipes\") which are stored in json and validated by
*starfish*. These parameters are then used to run *starfish* across the
complete dataset, in a cloud-based environment.

#### Output Data Formats

Image-based transcriptomics experiments emit three output formats that
contain identified features, all of which can be emitted in either
netcdf or zarr file formats, the latter being the format consumed by the
matrix service. In addition, it has the opportunity to emit image files
that have been processed to align images, remove autofluorescence and
enhance spots. All output formats (including SpaceTx image format)
contain full provenance records of the analysis environment, versions of
starfish and key dependencies, algorithms that were run, and their
parameters.

##### [[IntensityTable]{.underline}](https://spacetx-starfish.readthedocs.io/en/ajc-output-specifications/sptx-format/output_formats/IntensityTable/index.html)

Image-based transcriptomics experiments image the same tissue using
multiple color channels, and after subjecting the tissue to different
reagents. The physical locations of fluorescent molecules manifest as
spots in the images, and these spots are detected and aggregated across
fluorescence channels and time to build codes. In practice, the process
of detecting and decoding spots contains significant possibilities for
error, and the IntensityTable supports analysis and QC for this process.

##### [[SegmentationMask]{.underline}](https://spacetx-starfish.readthedocs.io/en/ajc-output-specifications/sptx-format/output_formats/SegmentationMask/index.html)

In order to assign the spots identified above to individual cells, the
locations of cells within the images must be identified. Compressed
segmentation masks are a compact way to represent these data, which
store, for each cell, the portion of the image that is attributed to
that cell.

##### [[ExpressionMatrix]{.underline}](https://spacetx-starfish.readthedocs.io/en/ajc-output-specifications/sptx-format/output_formats/ExpressionMatrix/index.html)

*starfish* combines spot calls from the IntensityTable with cell areas
in the SegmentationMask to produce expression matrices. These are saved
in zarr format and look identical to the matrices emitted by the Data
Analysis Service pipelines, with the exception that each cell is
additionally annotated with metadata that specify the x, y, and z
location of the data.

##### Processed Images

Image-based transcriptomics experiments suffer from several sources of
error and variation which are removed via image processing. Microscope
stages or tissue slices may shift as reagents are added and removed
across imaging rounds, or which spectral filters are swapped between
channels. In addition, reagents can non-specifically bind to cellular
material, causing autofluorescence. Prior to spot-calling, images can be
aligned, background removed, and spots computationally enhanced.
*starfish* can output these processed images, annotated with information
sufficient to re-do the processing, so that the images input to spot
calling can be evaluated by users. If emitted, processed images will be
generated in SpaceTx Format.

#### HCA Sample Metadata

The HCA metadata team has built a metadata specification draft for
imaging data that references data in SpaceTx-Format. The issue that
formed the draft can be found
[[here]{.underline}](https://github.com/HumanCellAtlas/metadata-schema/issues/368).

### Data and Metadata validation

#### Input Data validation

SpaceTx Format consists of a json file that specifies how to compose a
dataset from a set of two-dimensional images. As a result, it is
possible to rapidly validate that a file is complete and internally
consistent. In addition, SpaceTx Format stores per-chunk checksum
information, enabling the data to be checked for integrity. It is
possible to carry out additional, more data-specific checks, such as
that images are not blank, contain spots, or contain cells, however
these checks take increasing amounts of time.

OME and SpaceTx can provide a docker container that validates image
datasets with the highest level of stringency possible in a DCP-provided
\"maximum validation time\".

#### Output Data validation

Each of the output formats produced by SpaceTx can be stored in Zarr and
read (on-demand) by *starfish* and xarray. SpaceTx provides a docker
container that loads and validates each output file using the xarray
library. Validation can be tuned to occur within a DCP-provided target
validation time.

### Data Processing Workflows

To process data in the DCP, the *starfish* team can deliver WDL-wrapped
data processing workflows for any of the existing image-based
transcriptomics pipelines. These workflows are based on the *starfish*
python library, which in turn leverages standard open-source python
projects. *starfish* provides a docker container that exposes both a
python API and a CLI, and supports composable, modular, pipelines. These
pipelines take as inputs images in SpaceTx Format and a pipeline recipe.
Each assay emits output in the formats specified above. More detail on
the state of image-based transcriptomics analyses can be found
[[here]{.underline}](https://docs.google.com/document/d/18Ve9C_cEn5lI2paivfgTAyjsM62qMyUF_s4mj82l1L0/edit#).

### Data Processing Infrastructure

All *starfish* pipelines share the same inputs and outputs. It should
therefore be possible to deliver a single adapter WDL that satisfies any
SpaceTx assay, and a family of subscriptions with minimal inter-class
changes that specify the particular assay in question.

### Delivery of Output Data to Users

#### Matrices

As mentioned above, image-based transcriptomics output data adhere to
existing Matrix Service specifications and can be served \"as is\" for
the imaging prototype. Additional information on cellular geometries and
QC will be provided as a supplementary metadata zarr archive.

However, other imaging experiments will likely require a generalization
of the internal namespace of the DCP. Currently, the DCP standardizes
around GENCODE gene IDs and HGNC gene symbols. Image based proteomics
will require protein identifiers such as UNIPROT, and other use cases
that measure sub-cellular components will likely require more general
cellular component information (e.g. \"endoplasmic reticulum\"). These
use cases could be supported by switching identifiers from, e.g.
\"ACTB\" to \"HGNC:ACTB\" by prepending the namespace of the IDs being
used in a given assay.

### **Acceptance Criteria**

Except where listed (\"responsible team\"), the starfish and OME teams
will do the majority of project management and execution for the
following deliverables. They will coordinate and collaborate with DCP
partners (\"contact person\") to validate design correctness and
minimize code changes and their impact.

+-----------------+-----------------+-----------------+-----------------+
| **Component/Del | **Responsible   | **No earlier    | **Contact**     |
| iverable**      | Team/Service**  | than**          |                 |
|                 |                 |                 | **Person **     |
+=================+=================+=================+=================+
| HCA Metadata    | EBI             | Complete        | Z. Perova       |
| Draft           |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Test Datasets   | *starfish*      | Complete        | A. Carr         |
| Available       |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Submission      | *starfish*      | Q2 2019         | A. Carr         |
| Datasets        |                 |                 |                 |
| Available       |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Analysis        | Broad           | Q2 2019         | ?               |
| Pipelines       |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Analysis &      | Broad           | Q2 2019         | A. Carr         |
| Adapter         |                 |                 |                 |
| Pipelines       |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Lira            | Broad           | TBD             | S. Ehsan        |
| subscription    |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Data Analysis   | Broad           | Q2 2019         | R. Wang         |
| Service         |                 |                 |                 |
| *starfish*      |                 |                 |                 |
| Pipeline Tests  |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Specification   | *starfish*      | Complete        | J. Moore        |
| for input       |                 |                 |                 |
| format          |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Specification   | *starfish*      | Q1 2019         | A. Carr         |
| for output      |                 |                 |                 |
| format          |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Input           | *starfish*, OME | Q3 2019         | J. Moore        |
| validation tool |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Output          | *starfish*, OME | Q2 2019         | A. Carr         |
| validation tool |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Ingest support  | EBI             | ?               | ?               |
| for the nested  |                 |                 |                 |
| arrays of       |                 |                 |                 |
| channels        |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| [[Ability to    | ?               | ?               | ?               |
| describe files  |                 |                 |                 |
| that belong     |                 |                 |                 |
| together]{.unde |                 |                 |                 |
| rline}](https:/ |                 |                 |                 |
| /github.com/Hum |                 |                 |                 |
| anCellAtlas/met |                 |                 |                 |
| adata-schema/is |                 |                 |                 |
| sues/623)       |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| [[Upload and    | ?               | ?               | ?               |
| download of     |                 |                 |                 |
| subdirectories] |                 |                 |                 |
| {.underline}](h |                 |                 |                 |
| ttps://github.c |                 |                 |                 |
| om/HumanCellAtl |                 |                 |                 |
| as/data-store/i |                 |                 |                 |
| ssues/1885)     |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| DSS spatial     | UCSC            | ?               | B. Hannafious   |
| data bundle (or |                 |                 |                 |
| bund*less*)     |                 |                 |                 |
| specification   |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| DSS scale       | UCSC / DCP      | TBD             | B. Hannafious   |
| testing         |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Data Browser    | UCSC            | TBD             | H. Schmidt      |
| facets for      |                 |                 |                 |
| spatial data    |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Matrix service  | *starfish*      | TBD             | M. Kinsella     |
| integration     |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Upload Bundle   | ?               | ?               | P. Shah         |
| for 1M+ files   |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| Image file      | ?               | ?               | ?               |
| download        |                 |                 |                 |
| latency testing |                 |                 |                 |
+-----------------+-----------------+-----------------+-----------------+

### **Unresolved Questions**

-   Who are the intended beta users for imaging support and what
    > functionality is being targeted?

-   Are there other roadblocks for imaging support in the DCP?

-   What, if any, additional automation may be needed in the DCP to
    > support imaging?

-   What is the appropriate separation of metadata in the SpaceTx Format
    > and the HCA metadata schema?

-   How should imaging metadata be validated?

### **Drawbacks and Limitations**

Imaging formats and pipelines have not yet reached a level of
standardization across the community. As such, flexibility around
updating and upgrading formats and pipelines will be necessary.
Specifically:

1.  The development of a chunked image file format[^1] that supports
    > cloud workflows for data at the scale that will become common over
    > the next 10 years is in an early stage. Working prototypes from
    > genomics, imaging, astronomy, and geosciences exist and a process
    > is beginning to unify formats. However, this process will evolve
    > over time, and it will be necessary to upgrade the data,
    > potentially multiple times, as backwards-incompatible changes are
    > made. To mitigate this risk, the OME and *starfish* teams are
    > willing to be responsible for producing containerized scripts to
    > upgrade data when this is necessary, and to adhere to schedules
    > that limit the frequencies of these updates.

<!-- -->

2.  The analysis of image-based transcriptomics assays are not yet fully
    > automated, and therefore during the pilot phase, support of
    > imaging data will rely on user-supplied parameterizations of
    > *starfish*-based workflows. This will mean that initial datasets
    > may contain batch effects and be difficult to (absolutely) compare
    > to one another. However, while parameter selection decisions could
    > bias the absolute abundance of detected spots, the relative
    > abundances of spots within datasets should be consistent, and
    > presentations at recent single-cell genomics, keystone, and HCA
    > science meetings have demonstrated the utility of integrating
    > image data with single-cell RNA-seq data, and the fields appetite
    > to engage in these experiments. The *starfish* team sees a path
    > towards automating parameter selection for these assays, which can
    > be expedited by accumulating data in the DCP and building
    > community consensus around image data processing. This is a strong
    > argument for adding support now, and implies that there will be a
    > point in the future when these pipelines are updated, at which
    > point it will be necessary to reanalyze the data. If the
    > standardized pipelines are built with *starfish*, the *starfish*
    > team will commit to ensuring that those pipelines are DCP
    > compatible.

### **Prior Art and Alternatives**

A number of imaging formats and platforms exist that could provide parts
of an imaging solution for the HCA. A few have been listed below. Rather
than being direct alternatives though, they may ultimately be of
interest to consumers as extensions, portals, or conversion options.
Widening the scope of the HCA to support ingestion of additional formats
and to integrate other analysis and visualization platforms may be a
natural reaction to community demand.

The initial benefit of the SpaceTx format, library, and consortium,
however, is the on-going generation of initial datasets by individuals
already involved in the HCA, making it an ideal and low-cost prototype
for how imaging can work in the DCP. Additional options can be
prioritized based on the scientific "guardrails".

#### File Formats

-   [[OME-TIFF]{.underline}](https://www.openmicroscopy.org/site/support/ome-model/ome-tiff/specification.html)

-   [[N5]{.underline}](https://github.com/saalfeldlab/n5)

-   [[HDF5]{.underline}](https://www.hdfgroup.org/solutions/hdf5/)

-   [[Zarr]{.underline}](https://zarr.readthedocs.io)

#### Analysis Platforms/Pipelines

-   [[ImageJ]{.underline}](https://imagej.net)

-   [[ilastik]{.underline}](https://ilastik.org)

-   [[QuPath]{.underline}](https://qupath.github.io/)

-   [[Orbit]{.underline}](https://www.orbit.bio/)

-   [[CellProfiler]{.underline}](http://cellprofiler.org)

#### Visualization Platforms

-   [[IDR]{.underline}](https://idr.openmicroscopy.org/about/)

-   [[OMERO]{.underline}](https://www.openmicroscopy.org/omero)

-   [[Cytomine]{.underline}](http://www.cytomine.be)

-   [[PathViewer
    > (Commercial)]{.underline}](https://glencoesoftware.com/products/pathviewer/)

**External contributors**
-------------------------

### *starfish* team

*starfish* is a software library that enables data scientists to create
analysis pipelines for any image-based transcriptomics assay and defines
a file format that supports scalable cloud-based execution of these
pipelines. The *starfish* team at CZI is currently focused on supporting
the SpaceTx consortium, which consists of the core developers of
image-based transcriptomics assays. The SpaceTx consortium aims to
benchmark image-based transcriptomics assays by comparing each assay run
on the same tissue, from a shared probe set. It aims to answer the
question \"which assay should I choose, given tissue characteristics, or
the nature of my biological hypothesis?\".

### Open Microscopy Environment (OME)

OME maintains open-source software and format standards for the storage
and manipulation of microscopy data. The project's foundation is an open
metadata specification, the OME Data Model. Using this specification,
OME builds and releases: OME-TIFF, an open file format that combines the
popular, broadly supported TIFF format for storing binary pixel data,
with an OME-based XML metadata header; Bio-Formats, a software plug-in
that reads \>150 proprietary file formats and converts them to the
OME-based model; and OMERO, an enterprise image data management platform
that enables access, processing, sharing, analysis and where appropriate
publication of scientific image data and metadata. Motivated by arising
requirements including those of the HCA, OME is investigating a
cloud-compatible chunked file format for long-term archiving of imaging
data.

[^1]: A [[Long-Term
    Vision]{.underline}](https://docs.google.com/document/d/1wVYfBRL_omsnlWqUBc72kOm-GLdxPa97vBuV-OxyAcY/edit)
    document contains a fuller discussion on the needs of this format.
