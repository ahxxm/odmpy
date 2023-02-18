# odmpy

A simple console manager for OverDrive/Libby audiobook loans. Originally a python port of [overdrive](https://github.com/chbrown/overdrive), it now supports additional features such as adding of metadata such as chapters, merging of files, and downloading via [Libby](https://help.libbyapp.com/en-us/6103.htm).

Requires Python >= 3.7.

## Features

1. Downloads the cover and audio files for an audiobook loan, using a downloaded `.odm` loan or direct via [Libby](https://help.libbyapp.com/en-us/6103.htm), with additional options to:
   - merge files into a single `mp3` or `m4b` file
   - add chapters information into the audio file(s)
2. Return a loan
3. Renew a loan (Libby only)
4. Display information about an `.odm` loan file

## Install

```bash
# Install / Update to specific version
pip3 install git+https://git@github.com/ping/odmpy.git@0.7.0 --upgrade

# Install / Update from latest source
pip3 install git+https://git@github.com/ping/odmpy.git --upgrade --force-reinstall

# Uninstall
pip3 uninstall odmpy
```

## Usage

> ### ⚠️ Breaking
> 
> From version 0.7, the `--retry/-r` option has been moved to the base `odmpy` command (similar to `--timeout/-t`) for consistency
>  ```bash
>  # previously
>  odmpy dl --retry 3 "MyLoan.odm"
>  odmpy libby --retry 3
>
>  # now
>  odmpy --retry 3 dl "MyLoan.odm"
>  odmpy --retry 3 libby
>  ```

### General information
```
usage: odmpy [-h] [--version] [-v] [-t TIMEOUT] [-r RETRIES]
             {info,dl,ret,libby,libbyreturn,libbyrenew} ...

Download/return an OverDrive loan audiobook

options:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  -v, --verbose         Enable more verbose messages for debugging.
  -t TIMEOUT, --timeout TIMEOUT
                        Timeout (seconds) for network requests. Default 10.
  -r RETRIES, --retry RETRIES
                        Number of retries if a network request fails. Default
                        1.

Available commands:
  {info,dl,ret,libby,libbyreturn,libbyrenew}
                        To get more help, use the -h option with the command.
    info                Get information about a loan file
    dl                  Download from a loan odm file.
    ret                 Return a loan file.
    libby               Download audiobooks via Libby.
    libbyreturn         Return audiobook loans via Libby.
    libbyrenew          Renew audiobook loans via Libby.

Version 0.7.0. [Python 3.10.6-darwin] Source at https://github.com/ping/odmpy/
```

### Download via Libby

To download from Libby, you must already be using Libby on a [compatible](https://help.libbyapp.com/en-us/6105.htm) device. 

You will be prompted for a Libby setup code the first time you run the `libby` command. To get a code, follow the instructions [here](https://help.libbyapp.com/en-us/6070.htm). You should only need to do this once.

```
usage: odmpy libby [-h] [--settings SETTINGS_FOLDER] [--ebooks]
                   [-d DOWNLOAD_DIR] [-c] [-m] [--mergeformat {mp3,m4b}] [-k]
                   [-f] [--nobookfolder]
                   [--bookfolderformat BOOK_FOLDER_FORMAT] [--overwritetags]
                   [--tagsdelimiter DELIMITER] [--opf] [-r OBSOLETE_RETRIES]
                   [-j] [--hideprogress] [--direct] [--keepodm] [--latest N]
                   [--select N [N ...]] [--exportloans LOANS_JSON_FILEPATH]
                   [--reset] [--check]

Interactive Libby Interface for downloading audiobook loans.

options:
  -h, --help            show this help message and exit
  --settings SETTINGS_FOLDER
                        Settings folder to store odmpy required settings, e.g. Libby authentication.
  --ebooks              Include ebook (EPUB) loans. An EPUB (DRM) loan will be downloaded as an .acsm file
                        which can be opened in Adobe Digital Editions for offline reading.
                        Refer to https://help.overdrive.com/en-us/0577.html and 
                        https://help.overdrive.com/en-us/0005.html for more information.
                        An open EPUB (no DRM) loan will be downloaded as an .epub file which can be opened
                        in any EPUB-compatible reader.
  -d DOWNLOAD_DIR, --downloaddir DOWNLOAD_DIR
                        Download folder path.
  -c, --chapters        Add chapter marks (experimental).
  -m, --merge           Merge into 1 file (experimental, requires ffmpeg).
  --mergeformat {mp3,m4b}
                        Merged file format (m4b is slow, experimental, requires ffmpeg).
  -k, --keepcover       Always generate the cover image file (cover.jpg).
  -f, --keepmp3         Keep downloaded mp3 files (after merging).
  --nobookfolder        Don't create a book subfolder.
  --bookfolderformat BOOK_FOLDER_FORMAT
                        Book folder format string. Default "%(Title)s - %(Author)s".
                        Available fields:
                          %(Title)s : Title
                          %(Author)s: Comma-separated Author names
                          %(Series)s: Series
  --overwritetags       Always overwrite ID3 tags.
                        By default odmpy tries to non-destructively tag audiofiles.
                        This option forces odmpy to overwrite tags where possible.
  --tagsdelimiter DELIMITER
                        For ID3 tags with multiple values, this defines the delimiter.
                        For example, with the default delimiter ";", authors are written to the artist tag as "Author A;Author B;Author C".
  --opf                 Generate an OPF file for the book.
  -r OBSOLETE_RETRIES, --retry OBSOLETE_RETRIES
                        Obsolete. Do not use.
  -j, --writejson       Generate a meta json file (for debugging).
  --hideprogress        Hide the download progress bar (e.g. during testing).
  --direct              Process the audiobook download directly from Libby without downloading an odm file.
  --keepodm             Keep the downloaded odm and license files.
  --latest N            Non-interactive mode that downloads the latest N number of loans.
  --select N [N ...]    Non-interactive mode that downloads loans by the index entered.
                        For example, "--select 1 5" will download the first and fifth loans in order of the checked out date.
                        If the 5th loan does not exist, it will be skipped.
  --exportloans LOANS_JSON_FILEPATH
                        Non-interactive mode that exports loan information into a json file at the path specified.
  --reset               Remove previously saved odmpy Libby settings.
  --check               Non-interactive mode that displays Libby signed-in status.
```

There are non-interactive options available:

- Export loans information to a json file
   ```bash
   # export current loans in json
   odmpy libby --exportloans "path/to/loans.json"
   ```
- Download the latest N number of loans in order of checkout
   ```bash
   # download latest checked out loan
   odmpy libby --latest 1
   ```
- Download selected loans
   ```bash
   # download 3rd and 5th loans in order of checkout
   odmpy libby --select 3 5
   ```

### Return via Libby
```
usage: odmpy libbyreturn [-h] [--settings SETTINGS_FOLDER] [--ebooks]

Interactive Libby Interface for returning loans.

options:
  -h, --help            show this help message and exit
  --settings SETTINGS_FOLDER
                        Settings folder to store odmpy required settings, e.g. Libby authentication.
  --ebooks              Include ebook (EPUB) loans.
```

### Renew via Libby
```
usage: odmpy libbyrenew [-h] [--settings SETTINGS_FOLDER] [--ebooks]

Interactive Libby Interface for renewing loans.

options:
  -h, --help            show this help message and exit
  --settings SETTINGS_FOLDER
                        Settings folder to store odmpy required settings, e.g. Libby authentication.
  --ebooks              Include ebook (EPUB) loans.
```
### Download with an `.odm` loan file

[`.odm`](https://help.overdrive.com/en-us/0577.html) files are currently downloadable from your library's OverDrive site and are meant for use with OverDrive's [now legacy app](https://company.overdrive.com/2021/08/09/important-update-regarding-libby-and-the-overdrive-app/).

```
usage: odmpy dl [-h] [-d DOWNLOAD_DIR] [-c] [-m] [--mergeformat {mp3,m4b}]
                [-k] [-f] [--nobookfolder]
                [--bookfolderformat BOOK_FOLDER_FORMAT] [--overwritetags]
                [--tagsdelimiter DELIMITER] [--opf] [-r OBSOLETE_RETRIES] [-j]
                [--hideprogress]
                odm_file

Download from a loan file.

positional arguments:
  odm_file              ODM file path.

options:
  -h, --help            show this help message and exit
  -d DOWNLOAD_DIR, --downloaddir DOWNLOAD_DIR
                        Download folder path.
  -c, --chapters        Add chapter marks (experimental).
  -m, --merge           Merge into 1 file (experimental, requires ffmpeg).
  --mergeformat {mp3,m4b}
                        Merged file format (m4b is slow, experimental, requires ffmpeg).
  -k, --keepcover       Always generate the cover image file (cover.jpg).
  -f, --keepmp3         Keep downloaded mp3 files (after merging).
  --nobookfolder        Don't create a book subfolder.
  --bookfolderformat BOOK_FOLDER_FORMAT
                        Book folder format string. Default "%(Title)s - %(Author)s".
                        Available fields:
                          %(Title)s : Title
                          %(Author)s: Comma-separated Author names
                          %(Series)s: Series
  --overwritetags       Always overwrite ID3 tags.
                        By default odmpy tries to non-destructively tag audiofiles.
                        This option forces odmpy to overwrite tags where possible.
  --tagsdelimiter DELIMITER
                        For ID3 tags with multiple values, this defines the delimiter.
                        For example, with the default delimiter ";", authors are written to the artist tag as "Author A;Author B;Author C".
  --opf                 Generate an OPF file for the book.
  -r OBSOLETE_RETRIES, --retry OBSOLETE_RETRIES
                        Obsolete. Do not use.
  -j, --writejson       Generate a meta json file (for debugging).
  --hideprogress        Hide the download progress bar (e.g. during testing).
```

#### Unable to download odm files?

OverDrive no longer shows the odm download links for macOS 10.15 (Catalina) and newer.
There are many ways to get around this but the easiest for me was to use a 
[bookmarklet](https://support.mozilla.org/en-US/kb/bookmarklets-perform-common-web-page-tasks).
Follow the instructions in this [gist](https://gist.github.com/ping/b58ae66359691db1d08f929a9e57a03d)
to get started.

Alternatively, if you have switched over to Libby, `odmpy` now
supports direct from Libby downloads through the `libby` command.

```bash
# view available options
odmpy libby -h
```

### Return an `.odm`
```
usage: odmpy ret [-h] odm_file

Return a loan file.

positional arguments:
  odm_file    ODM file path.

options:
  -h, --help  show this help message and exit
```

### Information about an `.odm`
```
usage: odmpy info [-h] [-f {text,json}] odm_file

Get information about a loan file.

positional arguments:
  odm_file              ODM file path.

options:
  -h, --help            show this help message and exit
  -f {text,json}, --format {text,json}
                        Format for output.
```

### Examples

```bash

# Start the Libby interface to select an audiobook loan to download
# The `libby` command shares almost all of the download options as `dl`
# Example, downloads will be saved in MyLoans/
odmpy libby -d "MyLoans/"

# Download via Libby without generating the odm file
odmpy libby --direct

# Download via Libby your latest loan non-interactively
odmpy libby --direct --latest 1

# Return loans via Libby
odmpy libbyreturn

# Renew loans via Libby
odmpy libbyrenew

# Download a book via an odm file to MyLoans/
odmpy dl -d "MyLoans/" "MyLoans/Book1.odm"

# Return Book1.odm
odmpy ret "MyLoans/Book1.odm"

# Get information about a loan Book1.odm
odmpy info "MyLoans/Book1.odm"

```

`odmpy` also supports the reading of command options from a file. For example:

```bash
odmpy libby @example.dl.conf

odmpy dl @example.dl.conf MyLoans/MyBook.odm
```
where [`example.dl.conf`](example.dl.conf) contains the command arguments values

## Credits

- [overdrive](https://github.com/chbrown/overdrive)
- [pylibby](https://github.com/lullius/pylibby)

## Contributing

This repository uses [black](https://github.com/psf/black) to ensure consistent formatting.
The [CI Actions](https://github.com/ping/odmpy/blob/master/.github/workflows/lint-test.yml)
currently configured also include lint tests using [flake8](https://github.com/pycqa/flake8),
[pylint](https://github.com/PyCQA/pylint) and [mypy](https://github.com/python/mypy).

```bash
# 1. Install requirements for dev
pip3 install -r requirements-dev.txt --upgrade

# 2. Make changes

# 3. Check for linting errors
sh dev-lint.sh
```