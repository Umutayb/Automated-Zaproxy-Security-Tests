# Automated Zaproxy Security Tests
### Gitlab CI Integration Example
___
### Running Zaproxy on Gitlab CI:
Running Zaproxy on Gitlab CI is fairly easy, all it takes is to create a docker image, run it and pass a few arguments to it. 

The docker image can be built by using the official [Dockerfiles](https://github.com/zaproxy/zaproxy/tree/main/docker) provided by Zaproxy.

It's also possible to use the prebuilt docker image by Zaproxy, named [`owasp/zap2docker-stable`](https://hub.docker.com/r/owasp/zap2docker-stable/).

### How To Get Started:

First, a `.gitlab-ci.yml` file has to be created where the job configuration will be made. The official (or the customized
one you have created) Zaproxy docker images should be added to the job. Once added, executing a security test is simple.
Zaproxy provides a range of arguments to customize the tests that can be executed on a given website (baseline, full scan, api scan etc).

To execute a baseline test & generating an .xml report, the following script can be executed:
```shell
zap-baseline.py -t https://www.example.com -g gen.conf -x report.xml
```
If a different kind of test needs to be executed, it's sufficient to change the name of the first argument .py as in:

```shell
zap-api-scan.py -t https://www.example.com -g gen.conf -x report.xml
```
Different arguments to run different kinds of tests can be found [here](https://github.com/zaproxy/zaproxy/tree/main/docker).
If needed, Zaproxy can generate reports in different formats (JSON, HTML, XML & MD). This can be achieved by using different
arguments in the executing script, such as:
```shell
zap-baseline.py -t https://www.example.com -g gen.conf  -J report.json
```
Will create a JSON report. More info on baseline scans can be found [here](https://www.zaproxy.org/docs/docker/baseline-scan/).

For more information and instructions, please visit [here](https://www.zaproxy.org/docs/docker/about/).

It's also possible to execute a security test from your local using Dockerized Zaproxy. If Docker is already installed &
added to the path, running;
```shell
docker run -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py \
-t https://www.spritecloud.com/ -g gen.conf -x Report.xml
```
Will perform the tests & generate a report in the specified format. Finally, an example `.gitlab-ci.yml` would be:
```yml
image: owasp/zap2docker-stable
  variables:
    HOME_URL: https://www.example.com
  script:
    - zap-baseline.py -t $HOME_URL -g gen.conf -x Report.xml
```

___
###Calliope Pro Integration:
Zaproxy is great for security testing, but its reports can be hard to interpret.
Calliope Pro provides an interface where The Zaproxy results can be uploaded. The reports are then interpreted by Calliope which are then turned into easy to understand graphs. 

Calliope Pro can also be used to execute Gitlab CI jobs, from within its UI.
Uploading your reports to Calliope Pro is very easy:

- Go to [Calliope Pro](https://www.calliope.pro) & sign in (or sign up).

- Add your company & create a profile for your test.

- Find your profile id (`https://app.calliope.pro/profiles/3886` < profiles/profileID, 3886 in this case)

- Find your API Key by clicking your name (top right corner) and selecting personal information. There, click menu item called `Access tokens`

- With your profile id & api key, a `.gitlab-ci.yml` file can be created to get the job automatically upload test results to your profile.

A sample `.gitlab-ci.yml` file would be:
```yml
run-baseline-test:
  image: owasp/zap2docker-stable
  variables:
    PROFILE_ID: PROFILE_ID
    HOME_URL: https://www.spritecloud.com/
    API_KEY: API_KEY
  script:
    - mkdir /zap/wrk/
    - zap-baseline.py -t $HOME_URL -g gen.conf -x Report.xml
    - cp /zap/wrk/Report.xml .
  after_script:
    - curl -X POST "https://app.calliope.pro/api/v2/profile/$PROFILE_ID/import"  -H  "accept:application/json" -H "x-api-key:$API_KEY" -H  "Content-Type:multipart/form-data" -F "file[]=@/zap/wrk/Report.xml" -F "envelope=false" -F "smart=true"
```