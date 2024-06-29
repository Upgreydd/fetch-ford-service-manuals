# fetch-ford-service-manuals

Downloads HTML and PDF versions of Ford Service Manuals from PTS.

Bought a 72-hour subscription to Ford's service manuals and want to save it permanently?
Here's the repo for you.

These manuals are copyrighted by Ford, so don't share them!

## Table of Contents

- [Usage](#Usage)
- [Results (what do I get out of this?)](#results)
- [FAQ](#faq)

## Usage

Getting this to work currently requires some knowledge of browser DevTools.
If you're not sure how to use them, ask a friend who does.

This script uses [`playwright`](https://github.com/microsoft/playwright), a headless browser interop library, to save documents
as PDF files rather than raw HTML (this way files include images).

### (Avoid) Using Windows

While this script has been verified to work on Windows natively (see [issue #6](https://github.com/iamtheyammer/fetch-ford-service-manuals/issues/6)), it's recommended to run it in WSL. Running in WSL makes installing things like Git and Node far easier.

WSL is a way to run Linux (Ubuntu is recommended for this project) in tandem with Windows. It's far faster than a virtual machine but still uses the real Linux kernel. Learn more and see install instructions [here](https://learn.microsoft.com/en-us/windows/wsl/install).

### Set up Node (>16.3) and Yarn

1. Install Node.js 16.3 or newer (with `corepack`)
2. Run `corepack enable`

### Get code and dependencies

1. Clone this repository with `git clone https://github.com/iamtheyammer/fetch-ford-service-manuals.git`, and enter the repository's directory (likely with `cd fetch-ford-service-manuals`)
    - Previously cloned? Run `git pull` to get up to date!
    - If `git pull` does **not** say `Already up to date.`, run the next 2 steps to ensure your dependencies are up-to-date.
    - If you get an error while pulling, try running `git stash`, `git pull`, then `git stash apply` to un-stash your files.
2. Run `yarn` to download dependencies
3. Run `yarn playwright-setup` to download and set up Playwright

### Set up PTS

1. If you haven't, purchase a PTS subscription from [here](https://www.motorcraftservice.com/Purchase/ViewProduct). The 72 hour subscription is fine.
2. Once purchased, go to PTS: click [here](https://www.motorcraftservice.com/MySubscriptions), then click on your subscription title. ![how to open PTS](img/open-pts.png)
3. Once PTS opens, navigate to your car.
    - **Do not use your VIN.**
    - On the left, choose *By Year & Model*, then select your car's year and model.
    - Press GO once selected.

### Set up template files
1. In templates/ create a copy of cookieString.txt.template named "cookieString.txt" and clear the contents.
2. In templates/ create a copy of params.json.template named "params.json".

### **2003 or newer:** Get data for your car

**If your vehicle was made BEFORE 2003, use [these](#2002-or-older-get-data-for-your-car) instructions.**

This script requires some data about your car that's not available in the PTS GUI in order to fetch the correct manual.

1. Open DevTools, and navigate to the Network tab.
2. Click on the Workshop tab in PTS.
3. Filter for the one POST to `https://www.fordservicecontent.com/Ford_Content/PublicationRuntimeRefreshPTS//publication/prod_1_3_372022/TreeAndCover/workshop/32/~WSLL/{some numbers here}`. It should look similar to the request in [this photo](img/workshop-request.png).
4. Click on that request, and look at the sent form data (i.e. the payload).
5. Open [`templates/params.json`](templates/params.json), and copy information from that request into the values of the JSON `.workshop` field.
    - **Do not add fields. Only change values.**
    - Change the values to match. You probably won't need to change anything under the line break.
    - If you can't find the Book Title or Wiring Book Title, look in the query string parameters. **Do not leave them blank!**
6. Get your wiring data: follow instructions [here](#all-vehicles-get-wiring-data).

### **2002 or older:** Get data for your car

**If your vehicle was made IN 2003 or LATER, use [these](#2003-or-newer-get-data-for-your-car) instructions.**

1. Click on the Workshop tab in PTS.
   - You may see a couple manuals, often a "Workshop Manual" and a "Body Collision Repair Manual". You can use this application to download both-- just repeat the process for each manual, just remember to change the output path for each manual!
   - To proceed, click on either manual.
2. In the sidebar, right click on "Alphabetical Index", and click "Copy Link Address" (see [picture](img/pre-2003-index.jpg)).
3. Open [`templates/params.json`](templates/params.json), and change only:
    - `workshop.modelYear` to the year of your car
    - `pre_2003.alphabeticalIndexURL` to the URL you copied in step 2
    - The rest will be filled in later
4. Open DevTools in your browser.
5. Get your wiring data: follow instructions [here](#all-vehicles-get-wiring-data).

### **All Vehicles:** Get wiring data

1. Create a copy of `cookieString.txt.template` called `cookieString.txt` if you haven't already.
2. Clear the DevTools Network pane (click on the trash can or circle with a line through it)
3. Go to the Workshop tab in PTS. 
4. Filter for the one POST to `https://www.fordservicecontent.com/Ford_Content/PublicationRuntimeRefreshPTS//publication/prod_1_3_{datecode}/TreeAndCover/workshop/32/~WSLL/{some numbers here}`.
5. Go to the request headers and find the "Cookie:" entry.
6. Open your `cookieString.txt` file, clear the contents, and copy these cookies into the file. Add a semi-colon and space at the end of the file because we will be adding more cookies in a later step.
   - Do **not** include the name (`cookieString.txt` should **not** include `Cookie:`, for example.)
   - NOTE: In Firefox, you MUST enable the *Raw* toggle at the top right of Response Headers, then copy it from there. If you don't, you'll get an invalid character error when trying to fetch wiring diagrams.
7. Click the Wiring tab at the top of PTS.
8. Filter for the GET request to this URL: `https://www.fordservicecontent.com/Ford_Content/PublicationRuntimeRefreshPTS//wiring/TableofContent` (there are query params at the end, that's ok). It should look similar to the request in [this photo](img/wiring-request.png).
9. Copy the `environment`,`bookType`, and `languageCode` query params into `.wiring` in `params.json`.
   - **If your vehicle was made before 2003** (or if `WiringBookTitle` or `WiringBookCode` are missing), you may find these in another request to `https://www.fordtechservice.dealerconnection.com/wiring/TableOfContents` (with some query params at the end):
   - `booktitle` → `WiringBookTitle`
   - `book` → `WiringBookCode`
   - Use these two requests to fill in `params.json` as best as you can.
10. Save `params.json`.
11. Filter for the GET request to this URL: `https://www.fordtechservice.dealerconnection.com/wiring/TableOfContents` (there are query params at the end, that's ok).
12. Go to the request headers and find the "Cookie:" entry.
13. Copy the cookies from this request and append them to the end of your `cookieString.txt` file. 
   - Do **not** include the name (`cookieString.txt` should **not** include `Cookie:`, for example.)
   - NOTE: In Firefox, you MUST enable the *Raw* toggle at the top right of Response Headers, then copy it from there. If you don't, you'll get an invalid character error when trying to fetch wiring diagrams.
14. Save `cookieString.txt`.

### Download the manual!

To download the manual as PDFs, run `yarn start -c templates/params.json -s templates/cookieString.txt -o /directory/where/you/want/the/downloaded/manual/`. You should see output that looks like [this](img/example-output.png).

Make sure that the directory for the downloaded manual is empty-- it'll have lots of subfolders.

You can get more param information by running `yarn start --help`. Notably, `--saveHTML` will save `.html` files along with the `.pdf` files downloaded by default, and `--ignoreSaveErrors` will continue downloading manuals if an error is encountered, skipping the file with an error.

It can take a little while! On a fast computer with a fast internet connection, and, more importantly, a fast disk drive, over 15 minutes to download the manuals for the 2005 Taurus. Be patient!

Also, the resulting folder is pretty sizeable. The folder for the 2005 Taurus was about 300mb, and the F150 folder was a couple gigabytes.

## Results

This bot downloads the **entire** workshop manual and **all** wiring diagrams for the vehicle you set up.

### **All vehicles:** Wiring diagrams

Wiring diagrams will be in `outputpath/Wiring`. There's also a `toc.json` file with the table of contents for the wiring diagrams.

#### Connector Views & Component Location Charts (~2006 or newer)

If you have a `Wiring/Connector Views` folder, you've got a special file in there: `Connectors.csv`.
It tells you where to find every connector in the car, and where it is in the Component Location Charts.
Open it in Excel or Google Sheets to see the data. Here's a quick example:

| Connector ID | Connector                            | Connector Location Views Page Number | Grid Reference | Location in Vehicle |
|--------------|--------------------------------------|--------------------------------------|----------------|---------------------|
| C168A        | 10R80 Transmission (2.7L)            | 29                                   | F5             | Transmission        |
| C1840        | Line Pressure Control (LPC) solenoid | 34                                   | E8             | Inside transmission |

- Connector ID is the ID of the connector in the wiring diagrams.
- Connector is a description of the connector.
- Connector Location Views Page Number is the page number in the Connector Location Views section of the manual.
  (`Wiring/Connector Location Views`)
- Grid Reference is the grid reference on that page. You'll see a grid with letters on the side and numbers on the top.
  Use the letter as the row and the number as the column to find connectors.
- Location in Vehicle is a simple description of where the connector is in the vehicle.

### **2003 or newer:** Workshop manual

The folder structure in the output directory will mimic the structure on PTS, so if a file has a path like `1: General Information -> 00: Service Information -> 100-00 General Information -> About this Manual`, it will be in the folder `outputpath/1: General Information/00: Service Information/100-00 General Information/About this Manual.pdf`.

The `cover.html` file contains the book's cover and a table of contents laid out in bullet points. The tree of those bullet points directly maps to the file structure of the downloaded manual. Note that some characters are not allowed in file/folder names, so characters like slashes, colons, and more are replaced with dashes when saving.

The `toc.json` file contains the computer-readable table of contents, with the name mapped to the "document number", which is used to fetch the PDF.

#### Truncated filenames

Most operating systems limit filenames to 255 bytes (not 255 characters). For filenames over 200 characters (which are fairly rare), the downloader will truncate the name, then add ` (docID truncated)` onto the end.

If you're having trouble finding a document with a long name, search for it in `toc.json`, where it will be a key with a value. That value is the `docID` which will be in the filename.

### **2002 or older:** Workshop manual

Vehicles from 2002 or older have a different, harder-to-fetch structure in the manual, so this tool just uses the alphabetical index. This means the output is a bit different; you'll simply get a flat structure with all pages in the manual in the output folder you specified.

You can easily browse the manual by opening `outputpath/AA_Table_Of_Contents.html`-- all the links work except for the letters at the top.

There are also a few special files:

- `AA_Table_Of_Contents.html` is a special, processed table of contents where all the links work! Open it in your browser to navigate the manual.
- `AAA_alphabeticalIndex.json` is a JSON file with all the links in the alphabetical index. It's not as useful as the table of contents, but it's there if you need it. It's a result of the processing script.
- `AAA_originalTableOfContents.html` is the original table of contents, so the links don't work. It's there if you need it. It's a result of the processing script.

These files are prefixed with `AAA` so they appear at the top of the file list in most file browsers.

## FAQ

### Which vehicles does this work with?

All the ones I've tested. Just for fun, I tried:

- 1995 F-150
- 2002 Taurus
- 2006 Taurus/Sable
- 2008 Mustang
- 2020 MKZ
- 2022 F-150

All worked flawlessly!

### How can I support this project?

As Ford continues to change how manuals are accessed, this project requires continuous maintenance.

If this project was helpful to you, you can support this project on GitHub sponsors (click the "Sponsor" button at the top of the page), [buy me a coffee](https://buymeacoffee.com/iamtheyammer), or just share it!

### Why did you make this?

I wanted to have the manual for my car, and I bought the subscription hoping to download a PDF, so that's exactly what I did!

### Why do you fetch pages one-at-a-time?

Two reasons. Firstly, I don't want to DDoS Ford (they also have Akamai in front of this, and a ton of parallel requests would absolutely trigger it and get us blocked). Secondly, it was easier to code synchronously.
