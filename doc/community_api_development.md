## Araport Data And Microservices API Development

### Introduction

**ADAMA** (Araport Data And Microservices API) is a software framework that implements data federation strategy for the [Arabidopsis Information Portal.](https://www.araport.org/) The framework aims to facilitate the data and information exchange for Arabidopsis Community, Researches and Scientists.

Generally speaking, ADAMA is a builder of webservices. It provides the infrastructure necessary to publish a webservice, such as security, scalability, fault tolerance, monitoring, caching, etc. A developer of an adapter can concentrate just in the source and transformations of the data. The complete list of the webservices builders can be found [here.](https://adama-dev.tacc.utexas.edu/docs/adapters/index.html)

**[Prerequisite Software](#prerequisite-software)**

**[Environment Setup](#environment-setup)**

**[API Development](#api-development)**

**[API Deployment](#api-deployment)**

**[Validation Procedure](#validation-procedure)**

**[Appendix](#appendix)**

   - [Advanced Metadata Usage](#advanced-metadata-usage)
  
   - [API Troubleshooting Tips](#api-troubleshooting-tips)
  
  
##<a name="prerequisite-software"></a>Prerequisite Software
The third-party components and tools to develop web services community application programming interfaces.

### GitHub Account, and Git (version control system).

* [Creating GitHub Account](https://github.com/join)
	
* [Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git#Installing-on-Mac)

### A command line client for issuing HTTP requests
* [cURL](http://curl.haxx.se/)
* [Wget](https://www.gnu.org/software/wget/)

### Python 

You may refer to [Python Installation Reference](http://docs.python-guide.org/en/latest/starting/install/osx/). In addition, step-by-step installation procedure provided below.

**Note:**

Before installing Python, you’ll need to install GCC. GCC can be obtained by downloading XCode, the smaller Command Line Tools (must have an Apple account) or the even smaller OSX-GCC-Installer package.

**Note:**

If you already have XCode installed, do not install OSX-GCC-Installer. In combination, the software can cause issues that are difficult to diagnose.


* **Install XCode**

Run command below in the command line terminal

```
$ xcode-select --install
```

* **Install Package Manager - [Homebrew](http://brew.sh/)**

To install Homebrew, open Terminal or your favorite OSX terminal emulator and run

```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

The script will explain what changes it will make and prompt you before the installation begins. Once you’ve installed Homebrew, insert the Homebrew directory at the top of your PATH environment variable. You can do this by adding the following line at the bottom of your ~/.bash_profile file.

```
$ vi  ~/.bash_profile
export PATH=/usr/local/bin:/usr/local/sbin:$PATH
```
Source your bash profile to make changes for PATH variable effective.

```
$ source  ~/.bash_profile
```
* **Install Python using package manager**

```
$ brew install python
```
Validate the python installed

```
$ python --version
Python 2.7.10
```

* **Setuptools & Pip**

Homebrew installs Setuptools and pip for you.

**Setuptools** enables you to download and install any compliant Python software over a network. 
**pip** is a tool for easily installing and managing Python packages.

* **[Installing Virtual Environments](https://github.com/kennethreitz/python-guide/blob/master/docs/dev/virtualenvs.rst)**

A Virtual Environment is a tool to keep the dependencies required by different projects in separate places, by creating virtual Python environments for them. It keeps your global site-packages directory clean and manageable.

**virtualenv** is a tool to create isolated Python environments. virtualenv creates a folder which contains all the necessary executables to use the packages that a Python project would need.

We highly recommend using virtualenv tool to install any additional packages for your project.

Install virtualenv via pip:

```
$ pip install virtualenv
```

##<a name="environment-setup"></a>Environment Setup

To interact, and develop for Araport Data Microservces APIs (ADAMA) you would need a user account at [Araport](https://www.araport.org/), ADAMA API keys, and authentication token. You would need an authentication key to access core ADAMA services, deploy, and validate status of your data API.


*	**[Create a user account at Araport](https://www.araport.org/user/register)**

* **Set ADAMA environment variables**

```
$  vi  ~/.bash_profile
$ export ARAPORT=https://api.araport.org
$ export ADAMA=$ARAPORT/community/v0.3
$ export USERNAME=<your username at araport.org>
$ export PASSWORD=<your password>
```

Source your bash profile to make changes for environment variables effective.

```
$ source  ~/.bash_profile
```

* **Get an Authentication Token** 

You would need obtain a set of API keys using one-time action command. If you already have your API keys, skip to the next section. 

 * **Create API client**
	
	Now you will issue a request to the ADAMA server to create API client.
	
	Open your terminal, and run command:
	
```
$ curl -Lk -X POST \
     -u "$USERNAME:$PASSWORD"  \
     -d "clientName=my_cli_app" \
     $ARAPORT/clients/v2
```

You should receive a response returned by the ADAMA server (shortened by brevity)
```
{
    "message": "Client created successfully.",
    "result": {
        ...
        "consumerKey": "gTgpCecqtOc6Ao3GmZ_FecVSSV8a",
        "consumerSecret": "hZ_z3f4Hf3CcgvGoMix0aksN4BOD6",
        ...
    },
    "status": "success",
    ...
}
```

Save the fields `consumerKey` and `consumerSecret`. You would need them later to obtain an authentication token.

 * **[Obtain an Authentication Token](#authentication-token)**

Obtain an authentication token from the OAuth service (please, substitute `consumerKey` and `consumerSecret` with the values returned for your client application `my_cli_app`.

```
$ curl -Lk -X POST \
     -u "$CONSUMER_KEY:$CONSUMER_SECRET"  \
     -d "grant_type=password" \
     -d "username=$USERNAME" \
     -d "password=$PASSWORD" \
     -d "scope=PRODUCTION" \
     $ARAPORT/token
```
The returned response received from the OAuth service should look like:

```
{
    "access_token": "de32225c235cf47b9965997270a1496c",
    "expires_in": 14266,
    "refresh_token": "196f45495f6d9d0bc15416f7c55c39a",
    "token_type": "bearer"
}
```

Please, note the value of `access_token` token field. The `access_token` field is the value of the token you will need to use in your development wokflow: deploy your data API, validate the API status, or create an isolated environment on a remote server to interact with your API.

Open your terminal, and save the `access_token` field in the environmment variable `TOKEN`:

```
$export TOKEN=de32225c235cf47b9965997270a1496c
```
Look at your `TOKEN` variable:

```
$echo $TOKEN
de32225c235cf47b9965997270a1496c
```


##<a name="api-development"></a>API Development Workflow

Your development workflow would consist of several major steps:

* [Creating an isolated environment/namespace](#create-namespace) on a remote server
to interact with your data API (1)

The namespace allows your code run independently from apis developed by others. It is one of components that would uniquely identify the services provided by your API.

* [Structuring your API to create unique services](#api-structure) (2)

* [Documenting your API in a metadata description file](#describe-metadata) (3)

* [Coding your API](#code-api) (4)

* Testing API in a local envinronment (5)

* Deployment of API to a remote server in a development namespace (6)

* Testing API in a remote server in a development namespace (7)

* Publishing/Deployment of API to a remote server in a production namespace (8)

* Testing published API in a remote server in a production namespace (9)

You will find yourself performing steps (4) - (9) quite often until you're satisfied with the results.

####<a name="create-namespace"></a>Create Namespace

The recommended way to create/test/deploy your services is first to publish your service in a developer workspace.
Araport naming convention assumes that any developer workspace which ends in `*-dev` is a development namespace. The development namespace will not visible by default.

Let's create `testuser1-dev` development namespace.

#### 1. [Make sure you have active unexpired API token](#authentication-token).

Here the steps that automate the procedure to refresh an authentication token and save your time.

* Save `consumerKey`, and `consumerKey` in your bash profile to avoid repetitive copy-pasting.
* Add function that would update an authentication token

* Edit your bash_profile from the terminal, and copy-past the text below:

```
$ vi ~/.bash_profile
  export CONSUMER_KEY=<actualValueofConsumerKey>
  export CONSUMER_SECRET=<actualValueofConsumerSecret>
  
  function update_api_token(){
curl -Lk -X POST \
     -u "$CONSUMER_KEY:$CONSUMER_SECRET"  \
     -d "grant_type=password" \
     -d "username=$USERNAME" \
     -d "password=$PASSWORD" \
     -d "scope=PRODUCTION" \
     $ARAPORT/token
}
```
- Source bash profile to make changes effective

```
$ source  ~/.bash_profile
```
* Run in the terminal
 
```
$ update_token
```
Response Received:

```
{
    "access_token": "91584099511ffb3de947bcc3c67c55c",
    "expires_in": 14266,
    "refresh_token": "196f45495f6d9d0bc15416f7c55c39a",
    "token_type": "bearer"
}
```
Export an authentication token as environment variable, and validate the value:

```
$ export TOKEN=91584099511ffb3de947bcc3c67c55c 
$ echo $TOKEN
91584099511ffb3de947bcc3c67c55c
```
**NOTE:**

In the future steps, you will run only 3 commands:

```
$update_token

$export TOKEN=<access_token_value>

$echo $TOKEN
```

####  2. [Create Development Namespace](#create-namespace)

The rationality behind this step the following. 

First, by default  your development namespace will be active workspace. This will help you to avoid copy-pasting its value into a number of deployment commands.

Second, it will help you avoid accidental deployment into a production environment until you're really ready.

```
$vi ~/.bash_profile

export NS=testuser1-dev
```
Source bash profile to make changes effective, andvalidate value of the namespace variable:

```
$ source  ~/.bash_profile

$ echo $NS
testuser1-dev

```

Formal command to create your namespace. We will substitute `namespace-value` with the actual value when we actually run it.

```
 curl -Lk -X POST \
     -F name=<namespace-value> \
     -F description="Namespace description" \
     -H "Authorization: Bearer $TOKEN" \
     $ADAMA/namespaces
```

Run command from the terminal to create your namespace

```
 curl -Lk -X POST \
     -F name=testuser1-dev \
     -F description="Test User 1 Dev namespace" \
     -H "Authorization: Bearer $TOKEN" \
     $ADAMA/namespaces
```

You should receive the response with success status for your namespace: `testuser1-dev`

```
{
    "result": "https://api.araport.org/community/v0.3/testuser1-dev",
    "status": "success"
}
```

2.1 [Other Namespace Operations](#namespace-operations)

* [Retrieve all namespaces](#list-namespaces)

```
$curl -Lk -X GET \
     -H "Authorization: Bearer $TOKEN" \
     $ADAMA/namespaces
```

```
You will a long list of namespaces, including yours.

....
 {
            "description": "Test User 1 Dev namespace",
            "name": "testuser1-dev",
            "self": "https://api.araport.org/community/v0.3/testuser1-dev",
            "url": null,
            "users": {
                "araport/ibelyaev@carbon.super": [
                    "POST",
                    "PUT",
                    "DELETE"
                ]
            }
        }
 .....
```

* [Delete Existing Namespace] (#delete-namespace)

* Formal command:

```
$curl -kL -X DELETE -H "Authorization: Bearer $TOKEN" $ADAMA/<namespace_value>
```
Example (don't have to run):

```
$curl -kL -X DELETE -H "Authorization: Bearer $TOKEN" $ADAMA/testuser1-dev

{
    "status": "success"
}

```


###<a name="api-structure"></a>Structuring your API to create unique services

By now you have prepared your environment for the actual coding.

In this section we will:

* [Create API Directory Structure](#api-directory-structure)
* [Initialize Python Virtual Environment in each source folder](#init-virtualenv)
* [Create Initialization Files](#create-init-files)
* [Push your API in a GitHub Repository](#push-github). 

For example, you would like to develop a data API that provides current weather data.

You are planning to create two services:

* Weather Data by Geolocation
* Weather Data by Zipcode

####<a name="api-directory-structure"></a>Create API Directory Structure

You will create two sub-directories in your source root directory to clearly delineate your services.

Let's get started.

Go to your source code directory and create api root directory

```
$cd /software/git/scienceapps
$mkdir weather_api
$cd weather_api
$pwd
/software/git/scienceapps/weather_api

```
Initialize git repository:

```
$git init .
Initialized empty Git repository in /software/git/scienceapps/weather_api/.git/
```

Create source subdirectory to store weather by geolocation inside your source code root directory:

```
$cd /software/git/scienceapps/weather_api
$mkdir -p services/weather_by_geolocation
$cd /software/git/scienceapps/weather_api/services/weather_by_geolocation
```

Create source subdirectory to store weather by zipcode inside your source code root directory:

```
$cd /software/git/scienceapps/weather_api
$mkdir -p services/weather_by_zipcode
$cd /software/git/scienceapps/weather_api/services/weather_by_zipcode
```

####<a name="init-virtualenv"></a>Initialize Virtual Environmnet

Initialize virtual environment inside weather by geolocation subdirectory:

```
$cd /software/git/scienceapps/weather_api/services/weather_by_geolocation
$pwd
/software/git/scienceapps/services/weather_api/services/weather_by_geolocation
$virtualenv venv
New python executable in venv/bin/python2.7
Also creating executable in venv/bin/python
Installing setuptools, pip, wheel...done.
```
Activate your virtual environment inside your weather by geolocation directory:

```
$cd /software/git/scienceapps/weather_api/services/weather_by_geolocation
$pwd
/software/git/scienceapps/weather_api/services/weather_by_geolocation
$source venv/bin/activate
(venv)ibelyaev-osx:weather_by_geolocation ibelyaev$
```
Initialize virtual environment inside weather by zipcode
subdirectory:

```
$cd /software/git/scienceapps/weather_api/services/weather_by_zipcode
$pwd
/software/git/scienceapps/services/weather_api/services/weather_by_zipcode
$virtualenv venv
New python executable in venv/bin/python2.7
Also creating executable in venv/bin/python
Installing setuptools, pip, wheel...done.
```
Activate your virtual environment inside your weather by zipcode directory:

```
$cd /software/git/scienceapps/weather_api/services/weather_by_zipcode
$pwd
/software/git/scienceapps/weather_api/services/weather_by_zipcode
$source venv/bin/activate
(venv)ibelyaev-osx:weather_by_geolocation ibelyaev$
```

Now your virtual environment is activated for each service, and you can install additional python packages in the isolated environment.


####<a name="create-init-files"></a>Create Initialization Files

ADAMA framework requires the presense of empty __init__.py files in each service directory including root service api folder.

```
$ cd /software/git/scienceapps/weather_api/services/
$pwd /software/git/scienceapps/weather_api/services
$touch __init__.py
$ls
__init__.py		weather_by_geolocation	weather_by_zipcode
```
Repeat for each service:

* weather by geolocation
	
```
$ cd /software/git/scienceapps/weather_api/services/weather_by_geolocation
$pwd
/software/git/scienceapps/weather_api/services/weather_by_geolocation
$touch __init__.py
$ls
__init__.py
```
* weather by zipcode

```
$ cd /software/git/scienceapps/weather_api/services/weather_by_geolocation
$pwd
/software/git/scienceapps/weather_api/services/weather_by_geolocation
$touch __init__.py
$ls
__init__.py
```

* **Add Metadata Descriptor File for each service**:

* weather by geolocation

```
$ cd /software/git/scienceapps/weather_api/services/weather_by_geolocation
$pwd
/software/git/scienceapps/weather_api/services/weather_by_geolocation
$touch metadata.yml
$ls
__init__.py	metadata.yml
```

* weather by zipcode
	

```
$ cd /software/git/scienceapps/weather_api/services/weather_by_zipcode
$pwd
/software/git/scienceapps/weather_api/services/weather_by_zipcode
$touch metadata.yml
$ls
__init__.py	metadata.yml
```

The metadata descriptor files will be empty for now. You will edit them at later point.

* **Add Main Python Module File for each service**:

* weather by geolocation

```
$ cd /software/git/scienceapps/weather_api/services/weather_by_geolocation
$pwd
/software/git/scienceapps/weather_api/services/weather_by_geolocation
$touch main.py
$ls
__init__.py	main.py		metadata.yml
```

* weather by zipcode
	

```
$ cd /software/git/scienceapps/weather_api/services/weather_by_zipcode
$pwd
/software/git/scienceapps/weather_api/services/weather_by_zipcode
$touch main.py
$ls
__init__.py	main.py		metadata.yml

```

The main python module files will be empty for now. You will edit them at later point.

API Application Structure:

| Directory                                   | Comment                        | Content                |
|---------------------------------------------|--------------------------------|------------------------|
| weather_api                                 | Root API directory             | services               |
| weather_api/services                        | API Services directory         | __init__.py            |
|                                             |                                | weather_by_geolocation |
|                                             |                                | weather_by_zipcode     |
| weather_api/services/weather_by_geolocation | Weather by Geolocation Service | __init__.py            |
|                                             |                                | main.py                |
|                                             |                                | metadata.yml           |
| weather_api/services/weather_by_zipcode     | Weather by Zipcode Service     | __init__.py            |
|                                             |                                | main.py                |
|                                             |                                | metadata.yml           |

####<a name="push-github"></a>Push API in a GitHub Repository

* Create a GitHub Repository

	* **Login in GitHub, and create a remote Github Repository**

	So far, we have created [Weaher API GitHub Repository.](https://	github.com/Arabidopsis-Information-Portal/weather_api/tree/master)

	* **Syncronize your local repo with your remote GitHub Repository**
	
		* Copy Repository URL
		
		**NOTE:**
		Our examples uses git URL:
		
		```
		https://github.com/Arabidopsis-Information-Portal/weather_api.git
		```
		
		Generally, your git URL will look like:
		
		```
		<https://github.com/<YOUR_NAME>/<YOUR_REMOTE_REPO_NANE>.git>
		```
	  * Run commands in your terminal:
	  
	```
	$cd /software/git/scienceapps/weather_api
	# see remote repositories associated with a local repo
	$ git remote -v
	# add remote repo
	$git remote add origin https://github.com/Arabidopsis-Information-Portal/weather_api.git
	# add upstream tracking repo
	$ git remote add upstream https://github.com/Arabidopsis-Information-Portal/weather_api.git
	# add upstream tracking branch
	$git branch --set-upstream-to=origin/master master
	$git remote -v
	origin	https://github.com/Arabidopsis-Information-Portal/weather_api.git (fetch)
origin	https://github.com/Arabidopsis-Information-Portal/weather_api.git (push)
upstream	https://github.com/Arabidopsis-Information-Portal/weather_api.git (fetch)
upstream	https://github.com/Arabidopsis-Information-Portal/weather_api.git (push)
# git pull
	```
	
	* **Add your local changes, commit, and push**
		
```
$ cd /software/git/scienceapps/weather_api/services
$git status
$git add __init__.py
$git weather_by_geolocation/__init__.py
$git add weather_by_geolocation/main.py
$git add weather_by_geolocation/metadata.yml
$git add weather_by_zipcode/__init__.py
$git add weather_by_zipcode/main.py
$git add weather_by_zipcode/metadata.yml
$git status 
```

```
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   __init__.py
	new file:   weather_by_geolocation/__init__.py
	new file:   weather_by_geolocation/main.py
	new file:   weather_by_geolocation/metadata.yml
	new file:   weather_by_zipcode/__init__.py
	new file:   weather_by_zipcode/main.py
	new file:   weather_by_zipcode/metadata.yml
```

```
$git commit
$git push
```

```
Counting objects: 6, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 512 bytes | 0 bytes/s, done.
Total 5 (delta 0), reused 0 (delta 0)
To https://github.com/Arabidopsis-Information-Portal/weather_api.git
   635a8f2..5a52395  master -> master
```

##<a name="describe-metadata"></a>Describe API Metadata

ADAMA framework requires you provide the minimal metadata description file **metadata.yml** in order to successfully deploy the API, and provide the essential services such security, scalability, and fault tolerance.
In addition, it provides the discoverability for your API, online interaction with the deployed webservices, provenance and ability to completely document your RESTful API. It follows the [Open API Initiative](https://openapis.org/) to standartize how modern APIs are described. The [Swagger](http://swagger.io/]) Specification is a result of implementation of Open API Initiative, and let you fully describe, produce,  consume and visualize RESTful web services.

You can follow the [Introduction to Metadata Format](https://github.com/Arabidopsis-Information-Portal/adama/blob/master/docs/full/metadata.rst), tutorial how to document [API parameters](https://adama-dev.tacc.utexas.edu/docs/parameters/index.html). After you master the basics you may refer to the full [Swagger/Open API Specification](http://swagger.io/specification/) to learn more, and extend the base case as needed.

**IMPORTANT:**
The importance of the metadata file cannot be overstated. It **must** conform the YAML format, and pass the validation step during deployment. You would like to double check the conformance using the online  [YAML validators](http://codebeautify.org/yaml-validator) every single time you create/update the metadata file to avoid spurios error messages during deployment of your service.

The table below provides the explanation of a number of descriptors you may use to describe your service as well some additional comments/use cases when it is advisable to use them.

| Parameter        | Description                                                                                                                                                                                                                                                                                                                                                                                            | Requred ? | Example                                         |
|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------|-------------------------------------------------|
| name             | The name of the adapter                                                                                                                                                                                                                                                                                                                                                                                | yes       | weather_by_zipcode                              |
| description      | Free form text to describe the purpose of the adapter                                                                                                                                                                                                                                                                                                                                                  | yes       | Provides current weather by country and zipcode |
| version          | The version of the adapter. It supports [semantic versioning.](http://semver.org/)                                                                                                                                                                                                                                                                                                                     | yes       | 0.1                                             |
| type             | Type of the adapter. It supports one of the following: query, generic, map_filter, passthrough. The relevant discussion can be found [here.](https://www.araport.org/node/1235)                                                                                                                                                                                                                        | yes       | query                                           |
| url              | Url for the third party data source the adapter is accessing, if any. Depending on the type of the adapter, this may be for documentation purposes (query and generic), or it may be used directly by Adama to access the service on behalf of the user (map_filter and passthrough).                                                                                                                  | no        | http://api.openweathermap.org/                  |
| whitelist        | An additional list of ip's or domains that the adapter may need to access. May include additional instructions for the adapter how to process request headers, etc. Please refer to the [Advanced Metadata Usage to see more illustrative examples.](advanced-metadata-usage)                                                                                                                          | no        | api.openweathermap.org                          |
| main_module      | The name (including the path relative to the root of the git repository) of the main module for this adapter. If omitted, Adama will search for a module named main.*.                                                                                                                                                                                                                                 | yes       | services/weather_by_zipcode                     |
| notify           | An url that will receive a POST with the data of the new registered adapter once it is ready to receive requests.                                                                                                                                                                                                                                                                                      | no        |                                                 |
| requirements     | A list of extra modules to add to the adapter at installation time. These modules should be installable via the standard package manager of the language used by the adapter (for example: pip for Python, gem for Ruby, etc.). Please, do not include standard python libraries. For example, if you include json library (which standard for a python distribution) you will get a deployment error. | no        | requests                                        |
| validate_request | Whether to validate the parameters according to the provided documentation. By default this option is no. If enabled, the parameters of a request are validated before passing control to the user's code in the adapter.                                                                                                                                                                              | no        | no                                              |
| endpoints        | Documentation about the parameters accepted by this adapter [Documenting parameters.](http://adama.readthedocs.org/en/latest/adapters.html)                                                                                                                                                                                                                                                            | yes       | /search                                         |                      |

##<a name="code-api"></a>Code API Services

##<a name="api-deployment"></a>API Deployment

##<a name="validation-procedure"></a>Validation Procedure

##<a name="appendix"></a>Appendix

###<a name="advanced-metadata-usage"></a>Advanced Metadata Usage

###<a name="api-troubleshooting-tips"></a>API Troubleshooting Tips

## Common Commands used in Developer Workflow

## References

## <a name="references"></a>References

[ADAMA Documentation](http://adama.readthedocs.org/en/latest/)

### <a name="metadata-references"></a>Metadata Format

[Introduction to Metadata Format](https://github.com/Arabidopsis-Information-Portal/adama/blob/master/docs/full/metadata.rst)

[Documenting Metadata Parameters](http://adama.readthedocs.org/en/latest/parameters.html)

[Swagger Reference -  Metadata for RESTful API](http://swagger.io/)

[API Builders - ADAMA Adapters](http://adama.readthedocs.org/en/latest/adapters.html)

[YAML Validator](http://codebeautify.org/yaml-validator)

