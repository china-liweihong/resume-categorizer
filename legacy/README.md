Resume Categorizer
===================
Resume Categorizer extracts full name, e-mail address, phone number, and major from resumes and creates EXCEL file

Core Technology
--------------
1. Uses [Google Vision](https://cloud.google.com/vision/) engine for optical character recognition (OCR) on resumes to extract texts out of resumes
2. Uses [Stanford CoreNLP](https://stanfordnlp.github.io/CoreNLP/) and [Google NLP](https://cloud.google.com/natural-language/) engine for finding name entity
3. Uses [Clowder](https://clowder.ncsa.illinois.edu/) for data management system (DMS)

Tested on...
------------
- Ubuntu

Installation
------------
1. Install Python 2.x (tested with 2.7)
2. Set up a [virtual environment](http://docs.python-guide.org/en/latest/dev/virtualenvs/) for the Python
3. Boot up the virtual environment, and run `pip install -r requirements.txt` in the root of this project
4. Download Stanford CoreNLP Library and rename it to `stanford-corenlp`, then move the folder to the root of this project
5. [Set up the authentication for GCP](https://cloud.google.com/docs/authentication/getting-started) in your environment
  - Tested initially by setting an environment varialbe `GOOGLE_APPLICATION_CREDENTIALS` with the application credentials JSON file
6. Download Clowder server and rename it to `clowder`, then move the folder to the root of this project
  - Tested with the latest stable version (1.3.4)
  - Move the custom folder into clowder/
  - Set up [RabbitMQ](https://www.rabbitmq.com/download.html)
    - Install RabbitMQ
      - `sudo apt-get install rabbitmq-server` for Ubuntu
    - Get the latest Erlang (Follow the zero dependency Erlang RPM package for RabbitMQ)
    - To start RabbitMQ Management UI, `sudo rabbitmq-plugins enable rabbitmq_management`
    - [To allow guest/guest login](https://www.rabbitmq.com/access-control.html)
      - `sudo rabbitmqctl stop_app`
      - `sudo rabbitmqctl force_reset`
      - `sudo rabbitmqctl start_app`
      - `sudo service rabbitmq-server stop`
      - `sudo service rabbitmq-server start`
  - Set up MongoDB
    - Follow the [website](https://docs.mongodb.com/manual/tutorial/) for installation
    - To let mongodb server to start automatically, `systemctl enable mongod.service`
  - Set up JAVA to 1.8+
    - Follow the [instruction](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04) for the JAVA 1.8 installation
7. Get your Google credential and perform
  - `echo "path_to_json_google_crendential" >> ~/.bashrc`
  - Be careful not to use ~ for home directory; write out the full directory as it would be shown in `pwd`

Work Flow
------------
1. Run the Clowder
  - run `./clowder/bin/clowder`
2. Simultaneously, run the Stanford CoreNLP
  - run `./run_stanford_nlp.sh`
3. Simultaneously, run the resume extractor
  - in extractor folder, run `python resume_extractor.py`
4. Scan the front pages of all of resumes you want to categorize
5. Upload the stack to the Clowder
6. Sit back and wait for the engine to categorize all the resumes
7. Once the categorizing is over, you will see a link on the page of the uploaded resume stack

Default Configuration
-----------
- Port settings
  1. 9000: Clowder DMS
  2. 9001: Stanford Core NLP
  3. 15672: RabbitMQ management system

Tips for optimal results
-----------
1. Scan is preferred than photos taken by phone cameras
  - Tested on some phone camera PDFs and many of them were distorted
  - Scanning makes it easier for OCR engine to perform optimally
2. Scan just the first pages
  - There are some resumes that are more than one page (even Elen Musk has [one page resume](https://amp.businessinsider.com/images/5711368252bcd05b008bd03b-1536-2173.png))
  - Try to upload the first page (or the one page that matters)
  - Currently, this software does not support multiple pages

Things to be careful while running
-----------
1. Postfix issue
  - There was an issue with `org.apache.commons.commons-email-1.2.jar` not being able to send emails successfully
  - The problem was with Postfix in the Ubutnu machine
  - Fixed by following [this](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-as-a-send-only-smtp-server-on-ubuntu-16-04))
2. Port usability issue
  - There was Docker daemon running on the port 9000, causing Clowder to have an issue with running
  - I had to free the port 9000
  - Process:
    1. First check what is occupying the port
      - `sudo lsof -PiTCP -sTCP:LISTEN`
    2. If dockerd
      - `sudo systemctl stop docker`
    3. Check if it is gone
      - `sudo lsof -PiTCP -sTCP:LISTEN`
3. PDF text issue
  - Some resumes have texts embedded on their PDF files
  - For such PDFs, `convert` command (in /extractor/resources/process_pdf.sh) catches weird boxes
  - There needs to be stronger program to convert PDFs into image files
