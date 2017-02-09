# ucf-audit-docker

Dockerfile collection with different auditors for automated
accessibility evaluation.

## Build docker images

```
# Build AATT docker image
$ docker build -t docker-aatt -f ./docker-aatt/Dockerfile ./

# Build Asqatasun CLI docker image
$ docker build -t docker-asqatasun-cli -f ./docker-asqatasun-cli/Dockerfile ./docker-asqatasun-cli/

# Build Asqatasun service (with HTTP server) docker image
$ docker build -t docker-asqatasun-service -f ./docker-asqatasun-service/Dockerfile ./docker-asqatasun-service/

# Build Tanaguru service (with HTTP server) docker image
$ docker build -t docker-tanaguru-service -f ./docker-tanaguru-service/Dockerfile ./docker-tanaguru-service/

# Build UCF Accessibility Tool docker image
$ docker build -t docker-uat -f ./docker-uat/Dockerfile ./
```

## Usage


### AATT


```
# Start AATT docker container
$ docker run --name aatt -d docker-aatt

# Send a request with curl to AATT container
$ docker exec aatt curl -s -H "Content-Type: application/x-www-form-urlencoded" --data "source=<html></html>&output=json" http://0.0.0.0:3000/evaluate
[{"type":"error","msg":"A title should be provided for the document, using a non-empty title element in the head section.","code":"&lt;head&gt;&lt;/head&gt;","principle":"<a href=\"http://www.w3.org/TR/WCAG20/#operable\" target=\"_blank\">Operable</a>","techniques":"<a href=\"http://www.w3.org/TR/WCAG20-TECHS/H25\" target=\"_blank\">H25</a>"},{"type":"error","msg":"The html element should have a lang or xml:lang attribute which describes the language of the document.","code":"&lt;html&gt;&lt;head&gt;&lt;/h\nad&gt;&lt;body&gt;&lt;/body&gt\n&lt;/html&gt;","principle":"<a href=\"http://www.w3.org/TR/WCAG20/#understandable\" target=\"_blank\">Understandable</a>","techniques":"<a href=\"http://www.w3.org/TR/WCAG20-TECHS/H57\" target=\"_blank\">H57</a>"},{"errorcount":2}]
```

### Asqatasun

```
# Start Asqatasun CLI container
$ docker run --name asqatasun-cli -d docker-asqatasun-cli

# Start an audit at http://alterway.fr/
$ docker exec asqatasun-cli ./bin/asqatasun.sh -f /opt/firefox/firefox-bin -d 99 -r Rgaa30 http://alterway.fr/

Audit terminated with success at 2016-12-29 14:53:56.0
Audit Id : 2

RawMark : 47.83%
WeightedMark : 56.3611%
Nb Passed : 11
Nb Failed test : 12
[...]


# Start Asqatasun service container
$ docker run --name asqatasun-service -d docker-asqatasun-service

# Start an audit at http://alterway.fr/
$ docker cp scenario.json asqatasun-service:/root/asqatasun/
$ docker exec asqatasun-service curl -s -F "file=@scenario.json" http://0.0.0.0:3000/audit
/tmp/scenario.json
Audit terminated with success at 2016-12-29 14:59:34.0
Audit Id : 3

RawMark : 47.83%
WeightedMark : 56.3611%
Nb Passed : 11
Nb Failed test : 12
Nb Failed occurences : 58
Nb Pre-qualified : 37
Nb Not Applicable : 77
Nb Not Tested : 137

Subject : https://www.alterway.fr/
RawMark : 47.83%
WeightedMark : 56.3611%
Nb Passed : 11
Nb Failed test : 12
[...]

```

### Tanaguru

```
# Start Tanaguru service container
$ docker run --name tanaguru-service -d docker-tanaguru-service

# Start an audit at http://alterway.fr/
$ docker cp scenario.json tanaguru-service:$(docker exec tanaguru-service pwd)
$ docker exec tanaguru-service curl -s -F "file=@scenario.json" http://0.0.0.0:3000/audit
/tmp/scenario.json
Audit terminated with success at 2016-12-29 15:09:23.0
Audit Id : 5

RawMark : 52.63%
WeightedMark : 56.1909%
Nb Passed : 10
Nb Failed test : 9
Nb Failed occurences : 37
Nb Pre-qualified : 27
Nb Not Applicable : 33
Nb Not Tested : 180

Subject : https://www.alterway.fr/
RawMark : 52.63%
WeightedMark : 56.1909%
Nb Passed : 10
Nb Failed test : 9
[...]

```

### UCF Accessibility Tool (UAT)

```
# Start all services with docker compose
# (make sure you have built all images before)
$ docker-compose -f docker-compose/docker-compose.yml up -d


# Write file with url for AATT audit
$ docker-compose -f docker-compose/docker-compose.yml exec uat sh -c 'echo "consortium-projet-UCF/" > url.txt'

# Copy file with scenario for tanaguru and asqatasun audit
$ docker cp scenario.json "$(docker-compose -f docker-compose/docker-compose.yml ps -q uat)":/


# Start an audit with UAT and AATT
$ docker-compose -f docker-compose/docker-compose.yml exec uat uat --check-max-errors 2 --auditor aatt --file url.txt --webdriver-host wd --webdriver-port 4444 --auditor-host aatt --auditor-port 3000 --base-url http://www.ubiquitus-content-framework.fr/
Maximum accepted number of errors: 2
✖	5 errors	http://www.ubiquitus-content-framework.fr/consortium-projet-UCF/
✖	5 errors	http://www.ubiquitus-content-framework.fr/
$ docker-compose -f docker-compose/docker-compose.yml exec uat uat --check-max-errors 6 --auditor aatt --file url.txt --webdriver-host wd --webdriver-port 4444 --auditor-host aatt --auditor-port 3000 --base-url http://www.ubiquitus-content-framework.fr/
Maximum accepted number of errors: 6
✔	5 errors	http://www.ubiquitus-content-framework.fr/consortium-projet-UCF/
✔	5 errors	http://www.ubiquitus-content-framework.fr/

# To see details about AATT audit you can check the output files
$ docker-compose -f docker-compose/docker-compose.yml exec uat cat accessibilityResult-0.json
[
  {
    "type": "notice",
    "msg": "Check that the title element describes the document.",
    "code": "&lt;title&gt;Partenaires du\nprojet UCF - Projet\nUCF&lt;/title&gt;",
    "principle": "<a href=\"http://www.w3.org/TR/WCAG20/#operable\" target=\"_blank\">Operable</a>",
    "techniques": "<a href=\"http://www.w3.org/TR/WCAG20-TECHS/H25\" target=\"_blank\">H25</a>"
  },

[...]


# Start an audit with UAT and Asqatasun
$ docker-compose -f docker-compose/docker-compose.yml exec uat uat --check-min-mark 50 --auditor asqatasun --scenario scenario.json --auditor-host asqatasun --auditor-port 3000
Minimum accepted RawMark: 50%
✔	RawMark	59%
$ docker-compose -f docker-compose/docker-compose.yml exec uat uat --check-min-mark 70 --auditor asqatasun --scenario scenario.json --auditor-host asqatasun --auditor-port 3000
Minimum accepted RawMark: 70%
✖	RawMark	59%

# To see details about Asqatasun audit you can check the output files
$ docker-compose -f docker-compose/docker-compose.yml exec uat cat accessibilityResult.txt
Audit terminated with success at 2017-02-09 13:21:30.0
Audit Id : 2

RawMark : 59.09%
WeightedMark : 61.7778%
Nb Passed : 13
Nb Failed test : 9
Nb Failed occurences : 55
Nb Pre-qualified : 38
Nb Not Applicable : 77
Nb Not Tested : 137

Subject : https://www.alterway.fr/
RawMark : 59.09%
WeightedMark : 61.7778%
Nb Passed : 13
Nb Failed test : 9
[...]


# Start an audit with UAT and Tanaguru
$ docker-compose -f docker-compose/docker-compose.yml exec uat uat --check-min-mark 50 --auditor tanaguru --scenario scenario.json --auditor-host tanaguru --auditor-port 3000
Minimum accepted RawMark: 50%
✔	RawMark	65%
$ docker-compose -f docker-compose/docker-compose.yml exec uat uat --check-min-mark 70 --auditor tanaguru --scenario scenario.json --auditor-host tanaguru --auditor-port 3000
Minimum accepted RawMark: 70%
✖	RawMark	65%

# To see details about Tanaguru audit you can check the output files
$ docker-compose -f docker-compose/docker-compose.yml exec uat cat accessibilityResult.txt
Audit terminated with success at 2017-02-09 13:24:25.0
Audit Id : 6

RawMark : 65.0%
WeightedMark : 60.2079%
Nb Passed : 13
Nb Failed test : 7
Nb Failed occurences : 35
Nb Pre-qualified : 26
Nb Not Applicable : 33
Nb Not Tested : 180

Subject : https://www.alterway.fr/
RawMark : 65.0%
WeightedMark : 60.2079%
Nb Passed : 13
Nb Failed test : 7
[...]

```

# License

[Apache-2.0](LICENSE)
