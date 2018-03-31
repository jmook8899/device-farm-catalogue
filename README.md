# AWS Device Farm Catalogue

This project is a Jenkins Pipeline script that lists the available devices in AWS DeviceFarm and creates a catalogue by listing their most important characteristics and ARN's. 

The idea is to make it easy for testers to find the ARN's of the devices they want to test on when using AWS DeviceFarm.

[The resulting catalogue can be found here](./catalogue-table.md)

## Context

To run tests on AWS Device Farm, a tester must first create *Device Pools* which is easy if the tester has an access to the AWS Console.
If the tester do not have access to an environment and/or an automation of the device pool creation is needed, use the AWS CLI command line utility as an alternative.

To create a device pool, use the following syntax to configure selector rules in the payload of the [create-device-pool service](http://docs.aws.amazon.com/cli/latest/reference/devicefarm/create-device-pool.html):

```
[
  {
    "attribute": "ARN"|"PLATFORM"|"FORM_FACTOR"|"MANUFACTURER"|"REMOTE_ACCESS_ENABLED",
    "operator": "EQUALS"|"LESS_THAN"|"GREATER_THAN"|"IN"|"NOT_IN",
    "value": "string"
  }
  ...
]
```

Creating a device pool with rules based on platform, form factor or manufacturer can potentially result in a very large pool, which will also produce very long and expensive tests. Creating a device pool of just a few hand-picked devices is desireable and reasonable for small feature developments or on B2E apps where the device models used are well known. However, creating a device pool based on the an *IN ARN* rule...

```
[
  {
    "attribute": "ARN",
    "operator": "IN",
    "value": "[\"arn:aws:devicefarm:us-west-2::device:2DA046320998446B84407B50BFBD7757\",\"arn:aws:devicefarm:us-west-2::device:A4219155595B491287B7C959622A611C\"]"
  }
  ...
]
```

... can be cumbersome since there's no public AWS Device Catalogue which lists the ARN's.

The AWS Device Farm CLI command line utility does include a [list-devices service](http://docs.aws.amazon.com/cli/latest/reference/devicefarm/list-devices.html) which lists all available devices. 
After the testers pick the ARN's to test, they must create payloads for the *create-device-pool* service by hand.

This script makes finding those ARN's less painful by updating the catalogue periodically and listing it in an alphabetical order by the device's name.

## Implementation Notes

1. The script assumes you'll place it in a repository that's accessible to your Jenkins server.
2. The script assumes you've made AWS credentials available by using the [Jenkins Credentials Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Credentials+Plugin)
3. Make sure your password for Github doesn't contain '@'. If it does, the script will have no option but to url-encode it and Jenkins will write it encoded, but unmasked, to the log.
4. The script assumes you've generated SSH keys for the Jenkins user to access your github account.

* See [Generating Github SSH Credentials](https://help.github.com/articles/generating-an-ssh-key/)
* The SSH keys must be generated for your Jenkins user, so remamber to *sudo su - jenkins* before you generate the keys.
