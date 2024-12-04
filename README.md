[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

# ENA Webin-CLI Bulk Submission Tool
### Introduction
This tool is a wrapper to bulk submit read, un-annotated genome, targeted sequence or taxonomic reference data to the ENA using Webin-CLI.

The tool requires an appropriate metadata spreadsheet which it uses to generate manifest files for the user and validate or submit their submission. The tool does not handle study and sample registration, therefore visit [ENA Submissions Documentation](https://ena-docs.readthedocs.io/en/latest/submit/general-guide.html) for more information on this. The documentation also provides information on manifest file fields for your type of submission (which correlate to the headers in the spreadsheet file).

An example template spreadsheet has been provided (example_template_input.txt). This file is a tab-delimited text file, however the script also consumes spreadsheets in native MS Excel formats (e.g. .xslx) or comma-separated (.csv).

### Installation

#### Docker
To ease in usage, the tool has been containerised using [Docker](https://www.docker.com/). The only requirement is to have Docker [installed](https://docs.docker.com/get-docker/). Once installed, run the following commands to setup:

1. Clone the repository:
`git clone https://github.com/ebi-jlu8/ena-bulk-webincli.git && cd ena-bulk-webincli`
2. Build the docker image:
`docker build --tag ena-bulk-webincli .`
3. Ready to go! Run the tool using docker using the following command:
`docker run --rm -v <LOCAL_DATA_DIRECTORY>:/data ena-bulk-webincli -h` (for help)

`<LOCAL_DATA_DIRECTORY>` is recommended to be the `ena-bulk-webincli` directory on your local machine. In the example below, docker is used to submit reads to the test environment. The `/workdir` directory on the docker image is the working directory, containing the script, input spreadsheet and data files.

`docker run --rm -v pathto/ena-bulk-webincli:/workdir ena-bulk-webincli -u Webin-XXXX -p XXXX -g reads -s /workdir/example_template_read.txt -m submit -t`

Note: For data files to be submitted, relative file paths in accordance to `<LOCAL_DATA_DIRECTORY>` must be provided within the input spreadsheet. In the above example, the spreadsheet described a file `/workdir/data/Test_1.fq`, corresponding to the local file in `<LOCAL_DATA_DIRECTORY>/data/Test_1.fq`.

#### Singularity
In addition to the Docker container, a [Singularity](https://sylabs.io/guides/3.0/user-guide/index.html) container has also been generated to ease in setup and running the tool. To install Singularity, see their [Installation Guide](https://sylabs.io/guides/3.0/user-guide/installation.html#installation). Once installed,[build the singularity image](https://sylabs.io/guides/3.0/user-guide/build_a_container.html#building-containers-from-singularity-definition-files) using the definition file (`ena-bulk-webincli.def`):

1. Clone the repository:
`git clone https://github.com/nadimm-rahman/ena-bulk-webincli.git && cd ena-bulk-webincli`
2. Build the image:
`sudo singularity build ena-bulk-webincli.sif ena-bulk-webincli.def`
3. Ready to go! Run the tool using singularity with the following command:
`singularity run --bind <LOCAL_DATA_DIRECTORY>:/data ena-bulk-webincli.sif -h` (for help)

#### Other
To use the tool without a container:

1. Clone the repository:
`git clone https://github.com/nadimm-rahman/ena-bulk-webincli.git && cd ena-bulk-webincli`
2. Download the latest version of [Webin-CLI](https://github.com/enasequence/webin-cli/releases) installed.
3. Download tool dependencies listed below.
4. Run the tool using `python bulk_webincli.py --help`(for help).
5. The path to your downloaded `webin-cli.jar` file should be provided using the `-w [--webinCliPath]`.

The script accepts full paths to files (to be submitted e.g. fastq/fasta) within the input spreadsheet. To control location of outputs, a specific directory can be provided using the `--directory/-d` parameter, where the folders listed below will be generated.

### Usage

Mandatory arguments include Webin submission account username and password, genetic context and metadata spreadsheet. Note that the `--test/-t` flag can be specified to use Webin test submission services.

By default, the script utilises two additional directories:
1. 'manifests' - which houses all generated manifest files and report files.
2. 'submissions' - housing all validation and submission related reports and files, includes analysis and receipt XMLs of submissions.

##### Examples
Submitting reads to the test environment (sequential):

`python bulk_webincli.py -u Webin-XXXXX -p XXXXX -g reads -s INPUT_SPREADSHEET -m submit -t`

`docker run --rm -v localpathto/ena-bulk-webincli:/workdir ena-bulk-webincli -u Webin-XXXXX -p XXXXX -g reads -s /workdir/INPUT_SPREADSHEET -m submit -t`

`singularity run --bind localpathto/ena-bulk-webincli:/workdir ena-bulk-webincli.sif -u Webin-XXXXX -p XXXXX -g reads -s /workdir/INPUT_SPREADSHEET -m submit -t`
<p>&nbsp;</p>
Submitting genomes to the production environment (in parallel with 5 cores):

`python bulk_webincli.py -u Webin-XXXXX -p XXXXX -g genome -s INPUT_SPREADSHEET -m submit -pc 5`

`docker run --rm -v localpathto/ena-bulk-webincli:/workdir ena-bulk-webincli -u Webin-XXXXX -p XXXXX -g genome -s /workdir/INPUT_SPREADSHEET -m submit -pc 5`

`singularity run --bind localpathto/ena-bulk-webincli:/workdir ena-bulk-webincli.sif -u Webin-XXXXX -p XXXXX -g genome -s /workdir/INPUT_SPREADSHEET -m submit -pc 5`
<p>&nbsp;</p>
Validating reads, specifying an output directory (sequential):

`python bulk_webincli.py -u Webin-XXXXX -p XXXXX -g reads -s INPUT_SPREADSHEET -d OUTPUT_DIRECTORY -m validate`

`docker run --rm -v localpathto/ena-bulk-webincli:/workdir ena-bulk-webincli -u Webin-XXXXX -p XXXXX -g reads -s /workdir/INPUT_SPREADSHEET -d /workdir/OUTPUT_DIRECTORY -m validate`

`singularity run --bind localpathto/ena-bulk-webincli:/workdir ena-bulk-webincli.sif -u Webin-XXXXX -p XXXXX -g reads -s /workdir/INPUT_SPREADSHEET -d /workdir/OUTPUT_DIRECTORY -m validate`

### Dependencies
In addition to [Webin-CLI](https://github.com/enasequence/webin-cli/releases), the tool runs using [Python3.6+](https://www.python.org/downloads/) and requires installation of [Python Pandas](https://pandas.pydata.org/), [xlrd] (https://xlrd.readthedocs.io/en/latest/) and [joblib](https://joblib.readthedocs.io/en/latest/). This can be installed in a [virtual environment](https://docs.python.org/3/tutorial/venv.html). If using [Aspera](https://www.ibm.com/products/aspera/downloads) instead of FTP to upload files using Webin-CLI, ensure that you have downloadd Aspera and included it within your `$PATH`.
