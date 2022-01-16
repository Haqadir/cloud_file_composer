# Cloud File Composer
Reorganize a set of files into the ideal file size

### Objective
The problem we aim to solve is for composing files from either cloud storage or local storage to a specified size in order to meet end user requirements. Supported cloud providers include just GCP for the time being. 

2 cases:
1. files that are too large and need to be split
2. files that are smaller than ideal and need to be combined 

### What measures of size apply?
The main goal is to control size of files in terms of bytes. THere may be future improvements to handle measurements in terms of line count. 

### Inputs
- target size (int - bytes)
- cloud directory or local directory (string)
- header (boolean) 

### structure
- containerized solution for persistency
- deploys VM
- installs necessary SDKs and python packages
- authentications (Service accounts)
- download files to VM
- determine sizes of all files
- compares sizes of all files with desired output
    - groups files into 2 categories (larger smaller)
- ensures all files contain same header (requires header)
    - defaults header to digits 0-N, where N = number of commas in first line
    - ignores files with mismatched headers ( future improvement to provide feedback to user on exceptions due to mismatched headers)
- Send list of smaller files to 1 worker
    - if files are <40% of target, group files into numbers = ceiling(target_size/avg_size) 
    - send groups in sequence to another worker:
        - combine files together and upload to new cloud storage URL 
    - else ignore file and upload to new cloud storage URL 
- send list of larger files to different worker in parallel
    - call linux split function
    - upload files in parallel to cloud storage URL
- error handling for the above actions