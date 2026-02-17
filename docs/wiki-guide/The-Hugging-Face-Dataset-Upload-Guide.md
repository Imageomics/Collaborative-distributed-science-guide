# Hugging Face Dataset Guide

[Hugging Face](https://hf.co/) offers numerous methods for interacting with and creating datasets. This page provides a basic overview with some recommendations specifically targeting image dataset uploads, though the principles are transferrable to other data types. We list these options&mdash;in order of increasing complexity&mdash;with some guidance, recommendations, and links out to the appropriate parts of the Hugging Face docs for the most up-to-date information available.

1. [Web interface (UI)](#upload-a-dataset-with-the-web-interface): For smaller, simpler uploads.
2. [Hugging Face Command Line Interface (CLI)](#upload-a-dataset-with-the-hugging-face-cli): For most use-cases, easy access from cluster.
3. [Hugging Face API (python package)](#upload-a-dataset-with-hfapi): For when more fine-grained control than is achievable with the CLI is needed.
4. [Git/Git LFS](#upload-a-dataset-with-git): Main use-case is when multiple PRs lead to merge conflicts&mdash;Hugging Face provides no other means for resolution.

!!! info
    Some sections of the Hugging Face docs, such as for the `huggingface_hub`, has only version specific links for stable versions. In this case, if the link directs to an older version, there will be a banner to alert you to a newer version available, so keep an eye out for that updated version banner.

## Note on Authentication

All of these methods require authentication to edit datasets, ranging from passwords, to tokens, to SSH authentication, and all support editing **Public** (accessible to anyone on the internet) or **Private** (accessible only to members of the organization) repos. Two key notes on authentication:

1. Private repositories are only visible if you are authenticated.
2. If using tokens for access, be sure to create a [fine-grained token](https://huggingface.co/docs/hub/en/security-tokens#what-are-user-access-tokens), specifically for your needs.

## Upload a Dataset with the Web Interface

In the Files and versions tab of the Dataset card, you can choose to add file in the hugging web interface.

![Dataset repository Add file button](images/HF-dataset-upload/346190430-9e6cef9b-18ef-4d4a-84c5-1a3f75ac9336.png){ loading=lazy }

- Smaller files (<100MB), distributed, not well organized, less files


## Upload a Dataset with the Hugging Face CLI
- Bigger, more files, works from cluster
- THINK this is a wrapper on the datasets package
Which do you get on install??

## Upload a Dataset with HfApi

HF API (python package)
- Complex dataset structure, fine-grained control (not exposed on CLI) over how it shows up on HF
    - ex: glob pattern not clear enough for exclusion of subfolders or files


Please see the Hugging Face API docs for the most up-to-date guidance. For quick reference, to [upload by file](https://huggingface.co/docs/huggingface_hub/v1.3.4/package_reference/hf_api#huggingface_hub.HfApi.upload_file) or [upload by folder (structure maintained)](https://huggingface.co/docs/huggingface_hub/v1.3.4/package_reference/hf_api#huggingface_hub.HfApi.upload_folder).

``` py linenums="1"
from huggingface_hub import login

# Login with your personal token (find your tokens at: Settings/Access Tokens)
login()

from huggingface_hub import HfApi
api = HfApi()
repo_id = "<ABC-Center/dataset name>"

# Upload by file
api.upload_file(
    path_or_fileobj = <the local file path that you would like to upload>,
    path_in_repo = <the path in the repo>,
    repo_id = repo_id,
    repo_type = 'dataset'
)

# Upload by folder (maintain structure)
api.upload_folder(
    folder_path="/path/to/local/folder", # should end with folder containing data
    path_in_repo="path/to/folder/", # path desired for folder in repo
    repo_id= repo_id,
    repo_type="dataset",
    token_id = "paste-token-here" # if you're not logged in, HF does **not** recommend this method
)
```

## Upload a Dataset with Git

### Install Git LFS

Follow instructions at <https://git-lfs.com/>

### Install the Hugging Face CLI

```
brew install huggingface-cli
pip install -U "huggingface_hub[cli]"
```

### Enable the repository to upload large files

```
huggingface-cli lfs-enable-largefiles <your local dataset>
```

### Initialize Git LFS

```
git lfs install
```

### Track large or binary files (e.g., .jpg files)

Note that tabular data, such as CSVs, should **not** be tracked with `git lfs` unless they are too large to be tracked with `git`, i.e., if they are 100MB or larger (this is the [file size limit for GitHub](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github#file-size-limits)).

```
# Adds a line to .gitattributes, which Git uses to determine files managed by LFS
git lfs track "*.jpg"  
git add .gitattributes
git commit -m "Track large files with Git LFS"
```

#### Add, commit, and push the files

```
git add 
git commit -m 'comments'
git push
```


Repos can also be created through the Hugging Face API using the [create_repo method](https://huggingface.co/docs/huggingface_hub/v1.3.4/en/package_reference/hf_api#huggingface_hub.HfApi.create_repo) with the following parameters:

```py linenums="1"
repo_id = "<HF-Org-name>/<dataset-name>"
repo_type = "dataset"
private = True  # if you want the repo private
```

See also instructions using the [datasets package](https://huggingface.co/docs/datasets/create_dataset).





## Integrity Check

Sometimes uploads fail partway through, leaving one or more files un-uploaded. Unfortunately, it seems that there is not an easy solution to be alerted to these issues when not uploading through the UI. Additionally, using a glob pattern to set upload without a dry-run (in `git` terms, this would be running `git status` after adding files) can also lead to accidental exclusion. To catch these issues, we recommend the following integrity check after uploading a dataset[^1].

[^1]: Note that Hugging Face does impose [tiered rate limits](https://huggingface.co/docs/hub/rate-limits#rate-limit-tiers) (as of September 2025).

```python
import pandas as pd
from huggingface_hub import HfApi

api = HfApi()
repo_id = "<org-name>/<repo-name>"
repo_type = "dataset"

file_list = api.list_repo_files(repo_id=repo_id, repo_type=repo_type)
file_df = pd.Dataframe(columns = {"filepath": file_list})
metadata = pd.read_csv("path/to/metadata/file")

# assuming you use the same filepath in your system as in the repo
df = pd.merge(file_df, metadata, how = "inner", on = "filepath")
df.shape[0]  # this should match the number of expected images
```

!!! tip "Pro tip"
    If you don't have a metadata file for your images, use the [sum-buddy package](Helpful-Tools-for-your-Workflow.md#sum-buddy) to generate one in your local file system. This can also be used as a metadata file for the dataset viewer as needed (see [image datasets docs](https://huggingface.co/docs/hub/en/datasets-image) for more information on setting this up). Similar options are available for [audio](https://huggingface.co/docs/hub/en/datasets-audio) and [video](https://huggingface.co/docs/hub/en/datasets-video) datasets.
