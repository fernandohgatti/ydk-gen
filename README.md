# INSTALL

```
user-machine# cd <git_root>
user-machine# pip install -r requirements.txt
user-machine# source env.sh
```

## Install Tips:

Run Installation instruction under python virtualenv. Once you have virtualenv installed:

```
user-machine# virtualenv mypython
user-machine# source mypython/bin/activate
user-machine# pip install -r requirements.txt
```


# USAGE

```
user-machine# python generate.py --help
Usage: generate.py [options]

Options:
  --version           show program's version number and exit
  -h, --help          show this help message and exit
  --profile=PROFILE   Take options from a profile file, any CLI targets
                      ignored. Profile options override CLI currently
  -p, --python        Generate Python SDK
  -v, --verbose       Verbose mode
  --no-doc            Skip generation of documentation
  --output-directory  The output-directory . If not specified the output can be found under YDKGEN_HOME/gen-api/python
```

## Profile Approach

1. Construct a profile file, such as [```profiles/xr532-native-oc-bgp.json```](profiles/xr532-native-oc-bgp.json)

1. Generate the SDK using a command of the form:

```
python generate.py -p --profile profiles/xr532-native-oc-bgp.json
```

The generated SDK will in ```<git_root>/ydk/gen-api/python```.

### Profile File Documentation

A sample profile file is described below.

As should be fairly obvious, the file is in a JSON format. The initial section of metadata is mostly ignored for now. It will be used later.

```
{
    "version": "0.1.0",
    "Author": "Cisco",
    "Copyright": "Cisco",
    "Description": "Cisco IOS-XR Native Models From Git",
```

The "models" section of the file describes where to source models from. There are 3 sources:

- Directories
- Specific files
- Git, within which specific relative directories and files may be referenced

The sample below shows the use of git sources only.

```
    "models": {
        "git": [
```

We have a list of git sources. Each source must specify a URL. This URL should be one that allows the repository to be cloned without requiring user intervention, so please use a public URL such as the example below. There are three further options that can be specified:

- ```commitid``` - Optional specification of a commit in string form. The files identified will be copied from the context of this commit.
- ```dir``` - List of **relative** directory paths within git repository. All .yang files in this directory **and any sub-directories** will be pulled into the generated SDK.
- ```file```- List of **relative** file paths within the git repository.

Only directory examples are shown below.

```
            {
                "url": "https://github.com/YangModels/yang.git",
                "dir": [
                    "vendor/cisco/xr/532"
                ]
            },
            {
                "url": "https://github.com/YangModels/yang.git",
                "commitid": "f6b4e2d59d4eedf31ae8b2fa3119468e4c38259c",
                "dir": [
                    "experimental/openconfig/bgp",
                    "experimental/openconfig/policy"
                ]
            }
        ]
    },
```

## No Profile

1. Place a copy of the Yang data models ```<git_root>/ydk/yang```

1. Generate Python SDK:

```
user-machine# python generate.py -p --no-doc
```

The generated SDK will in ```<git_root>/ydk/gen-api/python```.

The same SDK may be generated using the profile [```profiles/models-in-repo.json```](profiles/models-in-repo.json), which also turns documentation generation **off**:

```
user-machine# python generate.py --profile profiles/models-in-repo.json
```


# Notes

Yang development kit:

- Tools that auto generate different programming language binding API's (Python, Ruby, GPB, Thrift, Objective C), Developers can use these objects/APIs, to write application
- Runtime libraries which provided "services" and transport code for App to talk to network devices (runtime for: Python, Ruby, gRPCServer). These runtime libraries also have protocol plugin, currently netconf plug has been added for testing.
- The runtime libraries have three parts:
    - Entity:  X object definations for YANG model. X here is programming language (Python, Ruby, Obj-C, GPBIDL, ThriftIDL etc)
    - NetworkElement:  Defines Network element instance (device object) with configuration of transport or protocol to speak to network device (e.g: netconf, restconf, gRPC client, etc.)
    - Services:
        - CRUDservice: Consume device object and Entity that does "encoding" and "transport" operation sepcific to "session". CRUD Service is enabled at three level (network-element level, entity level and each-attribute-in-entity level)
        - edService: Encode Decode service ONLY. This enables the encoding/decoding of entities into another format (e.g: entity<-->JSON/XML/GPB). 


# Python Notes

For Python entities and netconf session, CRUD service invoked on python class will:

- Encode python data objects to netconf XML payload
- Perform transport operation with device, collect the netconf response, 
- Decode netconf response in python class, return result to python app. 

> Note: The Python API currently supports just the CRUDService. This is internally written over encode/decode api.
edService is currently not open publicly



# Directory Structure

```
README          - install and usage notes
env.sh          - bash environment setup file
gen-api         - source dir or autogenerated SDK 
					- python (Python SDK)

generate.py     - bootstrap script to generate SDK for yang data models
yang            - yang models used by generate.py 
requirements.txt- python dependencies used during installation (refer README)
sdk             - sdk stubs
test            - test code, engg playground 
tools           - other tools leverage in ydk
```


# Running Unit Tests

```
user-machine# cd <git_root>
user-machine# cd ydk
user-machine# source env.sh
user-machine# python test/pygen_tests.py
```