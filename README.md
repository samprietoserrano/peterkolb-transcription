# Fraktur German/Peter Kolb transcription project

This repo contains code and output from my project at Stanford Center for Spatial and Textual Analysis (CESTA). The project consisted of transcribing German explorer Peter Kolb (or Kolbe)'s 1719 book on the Cape of Good Hope from scanned images to machine-readable and editiable text files. Versions of these files are in the `output-txt` folder. 

### Key Resources

* Source files for the book can be found at [InternetArchive.org](https://archive.org/details/bub_gb_MbxYAAAAcAAJ)
* The anthology article written by Sam about his work can be found at [cesta-io.stanford.edu](https://cesta-io.stanford.edu/anthology/2024-research-anthology/early-cape-travelers/)
* Sources of the archival texts used to build a historical German corpus for the spell checker: [DTA normalized/plaintext corpus 1600-1699](https://www.deutschestextarchiv.de/download#:~:text=b9a03d116c244c2da30ccc0937cc9c87-,normalized,-package), [DTA normalized/plaintext corpus 1700-1799](https://www.deutschestextarchiv.de/download#:~:text=b9a03d116c244c2da30ccc0937cc9c87-,normalized,-package), [CLARIN GeMiCorpus 1500-1700](https://llds.ling-phil.ox.ac.uk/llds/xmlui/handle/20.500.14106/2562)
* Guidance on using GCP Document AI can be found here: [product guide](), [scripting guide](https://github.com/GoogleCloudPlatform/python-docs-samples/tree/main/documentai/snippets), [enterprise OCR overview](https://cloud.google.com/document-ai/docs/enterprise-document-ocr)

## Summary

In this project I was tasked with taking Kolb's 1719 book --the most-thorough and cited of its kind in being a primary source of the experiences and perspectives of early European explorers and settlers in the Cape of Good Hope, from scanned images of the original print into textual files that would be compatible with modern research and, specificially, digital humanities methods. This presented the need for having the text in machine-readable form.

### Obstacles 
The two main challenges brought by the book were: 
1. understanding early-modern Fraktur German writing,
2. recognizing the reading order between columns and headers.

Most of the readily-accessible tools I explored still came up short in some way. 
| Tool  | Issues |
|-------|-------|
| all pre-built transcription web apps| failed to recognize the different column regions|
| Gemini, Chat-GPT-4o| Issue 2|
| Tool 3| Issue 3|

* all pre-built transcription web apps failed to recognize the different column regions;
* large-language models (LLMs) like Gemini or Chat-GPT-4o did not consistently produce quality output when prompted on pages of identical layout;
* and, even though they succeeded in region recognition, standard Python text extraction packages (ie. Tesseract)  lacked the language training to understand the Fraktur German print.

Along the way I considered training a custom model or text extraction porcessor, however I weighed the scope of the project and intended outcome and decided against the effort to manually tag and create training and validation sets. The focus of my task, and purspose of the larger Early Cape Travelers research project, was not to focus developing high accurary text tools but rather produce the best possible version of the Kolb book within my internship time. As I continued experimenting with tools, I was able to devise methods that I could apply to at least part of the text and thus utilize the most effective tools available in the most efficient combination.

After grouping all pages by layout, I arrived at two big groups: about 750 pages had two columns of text with minor outer-margin text (Group A), and about 250 pages of varying layouts with columns, tables, or images (Group B). For Group A, I was able to write a Python script for Google Cloud Platform’s “Vertex Vision AI” API. In my script, I take each page as an image, crop it into regions using predetermined coordinates, pass the regions to the Vision AI processor, and I get text output in the same order as the regions. For Group B, which had layouts too complex to crop into regions systematically, I used the Transkribus transcription platform. Within Transkribus, I manually created ‘bounding boxes’ over every text region I deemed necessary, ran a Transkribus pre-trained LLM, and then edited or organized the text around the model’s errors. 

Next, I moved into post-processing the extracted text with a handful of NLP open-source software. After smaller corrections, I faced the need to spell check the 550k words in the corpus. I found that all other spell checking tools seemed to not handle the historical vocabulary, I ended up creating and feedomh my own dictionary of German words from 1500-1800 to PySpellChecker. 

Finally, I was able to bring all text, images, and tables together into a Docx document that is readable, editable, and searchable for specific content depending on the research goals. In the `output-txt` there is also the plain text files for every page.

### Script pipeline

Below you find the order in which I utilized the scripts in this repo during my text processing pipeline:

1. `text-extraction\jpeg_conversion.py`
   * convert jp2 files (from Internet Archive) into JPEG
2. `text-extraction\extraction_gcp.py`
   * run the GCP processor on the ‘main’ page group JPEG; save the txt files
3. `text-extraction\pdf_splitter.py`
   * create subset PDF for each page group, process on Transkribus
5. `text-extraction\jpeg_duplicator.py`
   * copy JPEGs into folder for each page group, process on Transkribus
7. `text-processing\reindex_txt_names.py`
   * rename Transkribus-exported groups to be 1-indexed (not 0-indexed)
9. `text-processing\pp_maingroup.py`
   * post-process the ‘main’ page group with level 1 and level 2
11. `text-processing\pp_index.py`
    * post-process the ‘index’ page group in three levels (basic, non-special, specials)
13. `text-processing\pp-othergroups.py`
    * post-process all other page groups in two levels (line numbers, correct lines)
    * do each page group at a time; customize script paths
15. `text-correction\unhyphenate.py`
    * unhyphenate txt files as prep for running spellchecker
17. `text-correction\spellchecker.py`
    * run spellchecker program
    * run one of the corpus creation options first (to feed the spell checker)
    * once a corpus exists, run a spellchecker option with either hard-coded or fed input/output paths
19. `text-processing\page_order_mapmaker.py`
    * create a mapping file for each page group
21. `text-processing\page_order_mapreader.py`
    * read each mapping file and copy each page group into a single ‘merged’ folder
23. `text-compilation\create_docx.py`
    * create simple/format versions of compiled Docx for each non-blank page
25. `text-correction\manual_check.py`
    * optionally, run a manual review on unknown words
27. `text-compilation\manualcheck_update.py`
    * optionally, update the list of unknown words after a manual review
29. `text-correction\manualcheck_corpus.py`
    * optionally, run a manual review on the list of corpus words
    * this script auto-updates, thus no need for separate update script
   

### Page Group Breakdown
If interested in knowing what pages I group with what, below see the page numbers for each group. These page numbers are based on their PDF page number from the source PDF.

For example, the first page of the book has page number 1 (not 0), it would belong to the "pg_img-new" list from below, and its txt file would be 01.txt. Except that page is a pure image page with no text, so there was not a txt file created for it. 

Note that the page group "main" would consist of every page NOT in one of the lists below. 
~~~
group_lists = {
    "pg_begin": [i for i in range(9, 23)],
    "pg_toc": [23, 24, 25, 26, 27],
    "pg_starts": [29, 31, 32, 36, 46, 55, 56, 68, 86, 105, 121, 136, 151, 165, 212, 231, 256, 270, 281, 304, 317, 330, 341, 347, 364, 391, 409, 420, 444, 450, 482, 488, 500, 509, 516, 528, 543, 554, 562, 579, 586, 606, 621, 629, 634, 644, 653, 666, 683, 727, 728, 729, 735, 744, 760, 798, 807, 817, 830, 840, 849, 855, 863, 873, 881, 887, 904],
    "pg_tables": [404, 405, 406, 407, 408],
    "pg_index": [i for i in range(905, 985)],
    "pg_other": [706, 874],
    "pg_dict": [985, 986],
    "pg_skip": [58],
    "pg_img": [8, 76, 140, 170, 177, 192, 201, 210, 218, 236, 240, 456, 473, 491, 519, 523, 529, 557, 569, 576, 590, 600, 626, 647, 718],
    "pg_img-new": [1, 2, 5, 8, 76, 140, 170, 177, 192, 201, 210, 218, 236, 240, 456, 473, 491, 519, 523, 529, 557, 569, 576, 590, 600, 626, 647, 718],
    "pg_blank1": [i for i in range(1, 8)],
    "pg_blank2": [10, 18, 28, 77, 141, 171, 176, 193, 200, 211, 219, 237, 241, 474, 492, 520, 524, 530, 558, 570, 575, 589, 599, 625, 648, 719, 987, 988, 989, 990, 991, 992]
}
~~~

## Recommended Next Steps
* Run the manual check program with a German-speaker to vet unknown words and then run the updating version of the script.
* Continue expanding the corpus collection to add to the three file sets used.

#### Warnings
* Some pages that hold illustrations also contain some minor text, serving as descriptor of the illustration. However, of these pages, only page 17 (0017.txt) was run through the text extraction process (and oversight on my part). Future versions of this project should process all pages in the "pg_img-new" group that also have text.
* The source file from Internet Archive is **NOT** complete. There is missing content around page number 57-59, as seen by how the printed page numbers at the top of the page margins do not line up in sequence. In my work I decided to skip page 58.
