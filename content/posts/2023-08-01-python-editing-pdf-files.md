---
title: "Using Python to Edit PDF Files"
date: 2023-08-01T16:00:00-05:00
type: post
author: James
categories:
  - Software Development
tags:
  - Python
  - PDF
comments: true
---

I had a need to edit a batch of PDF files to remove some sensitive information such as account numbers and replace them with asterisks (*). When looking at some of the programs that are available to edit PDF's they seemed to be mostly paid programs that didn't have good batch options.

Writing a program in Python seemed like an interesting option and I did a quick survey of some of the existing packages. PyPDF2 looked to be a good library and had some examples on the web on how to do something similar, especially this particular gist: [Redact phrases of text from a PDF](https://gist.github.com/rileypeterson/723a8650affec02098fd5146f47bf488).

The example given in the gist worked with an old version of the PyPDF library (1.26.0). With some slight modifications could be made to the lastest version 3.0.1, as mentioned in the comments for the gist.

The final program worked well and can process a batch of files by specifying the folder that contains the PDF as an argument, e.g.

{{< highlight "bash" >}}
python edit-pdf pdfs
{{< / highlight >}}

The program will then do the following:

1. Read the local config file *substitutions.csv* and determine which strings will be replaced and what the replacement will be.
2. Loop through all of the files in the given folder to process each one.
3. For each file read the number of pages.
4. Loop through each page and get the content data (using an override version of the get_data function that does replacements).
5. Replace the content by merging the updated page with the old one.
6. Write the new content as a new PDF file in a subfolder called "edited"

The override function described above is implemented in a special class:

{{< highlight "Python" >}}
class EncodedStreamObject(StreamObject):
{{< / highlight >}}

The details of the class can be found in the code at the Github link below.

The override class is then instantiated in the following line:

{{< highlight "Python" >}}
# Override with replacement version
PyPDF2.generic._data_structures.EncodedStreamObject = EncodedStreamObject
{{< / highlight >}}

The code is available on Github: [edit-pdf.py](https://github.com/turnkey-commerce/edit-pdf/blob/master/edit-pdf.py)

Information on installation and running the code: [README](https://github.com/turnkey-commerce/edit-pdf/blob/master/README.md)

## Other Possible Uses

The program isn't limited to just redacting sensitive material with asterisks. It can also be used to replace incorrect information such as an address or other information.

These values can be specified in the CSV file that is used to define the replacements with the old_value, new_value inputs, e.g.

{{< highlight "csv" >}}
6364625, *******
1455 Girard St, 2325 Girardo St
{{< / highlight >}}
