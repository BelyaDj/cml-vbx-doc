# Rally - Refstack

[TOC]

---

## RefStack Overview

RefStack intends on being THE source of tools for interoperability testing of OpenStack clouds.
RefStack provides users in the OpenStack community with a Tempest wrapper, refstack-client, that 
helps to verify interoperability of their cloud with other OpenStack clouds. It does so 
by validating any cloud implementation against the OpenStack Tempest API tests.

Links:

* <https://refstack.openstack.org/#/about>
* <http://www.openstack.org/brand/interop/>
* <https://github.com/openstack/defcore>

---

## RefStack Client

Refstack-client is a command line utility that allows you to execute Tempest
test runs based on configurations you specify.  When finished running Tempest
it can send the passed test data to a RefStack API server.

#### Requirements for using Refstack-client

* Management-server with installed kvm virtualization
* installed Fuel-server as virtual machine in Management-server
* ready OpenStack deployment (OpenStack >= Kilo)
* access from Fuel to specific OpenStack Management LAN
* access from Fuel to internet
* ssh access to Fuel-server with root permissions
* "operator machine"
* git

#### Installation Refstack-client

1. Connect as root (or as sudo user) via ssh to Fuel-server 
2. Get the refstack client:

        git clone https://github.com/openstack/refstack-client

3. Go into the refstack-client directory:    

        cd refstack-client

4. Run the "easy button" setup:      

        ./setup_env

#### Usage

1. Prepare a tempest configuration file that is customized to your cloud environment
    (<http://docs.openstack.org/developer/tempest/configuration.html>)

    Or you can use rally-generated configuration file. For example: 

        /root/.rally/tempest/for-deployment-[id]/tempest.conf

2. Go into the refstack-client directory

        cd ~/refstack-client

3. Source to use the correct Python environment

        source .venv/bin/activate

4. Validate your setup by running a short test

        ./refstack-client test -c <Path of the tempest configuration file to use> -v -- --regex tempest.api.identity.admin.v2.test_roles

    or

        ./refstack-client test -c <Path of the tempest configuration file to use> -v -- --regex tempest.api.identity.v2.test_token

5. Run tests

    To run the entire API test set:
       
        ./refstack-client test -c <Path of the tempest configuration file to use> -v
       
    To run only those tests specified in a DefCore defined test file:
    
        ./refstack-client test -c <Path of the tempest configuration file to use> -v --test-list <Path or URL of test list>
    
    For example:
       
    run 99 tests (only the test cases required by the 2016.01 guidelines that have not been flagged.)             
           
        ./refstack-client test -c  ~/.rally/tempest/for-deployment-[id]/tempest.conf -v --test-list \
        "https://refstack.openstack.org/api/v1/guidelines/2016.01/tests?target=platform&type=required&alias=true&flag=false"
           
    or run 128 tests
               
        ./refstack-client test -c  ~/.rally/tempest/for-deployment-[id]/tempest.conf -v --test-list \
        "https://refstack.openstack.org/api/v1/guidelines/2016.01/tests?target=platform&type=required&alias=true&flag=true" 
                 
    You can choose tempest list here <https://refstack.openstack.org/#/guidelines> ('Test list' button)
    
    !!! note
        * Adding the `-v` option will show the Tempest test result output.        

        * Adding the `--upload` option will have your test results be uploaded to the
            default RefStack API server or the server specified by `--url`.

        * Adding the `--test-list` option will allow you to specify the file path or URL of
            a test list text file. This test list should contain specific test cases that
            should be tested. Tests lists passed in using this argument will be normalized
            with the current Tempest evironment to eliminate any attribute mismatches.        
        * Adding the `--url` option will allow you to change where test results should
            be uploaded.      
        * Adding the `-r` option with a string will prefix the JSON result file with the
            given string (e.g. `-r my-test` will yield a result file like
            'my-test-0.json').        
        * Adding `--` enables you to pass arbitrary arguments to the ostestr runner.
            After the first `--`, all other subsequent arguments will be passed to
            the ostestr runner as is. This is mainly used for quick verification of the
            target test cases. (e.g. `-- --regex tempest.api.identity.v2.test_token`)       
                
        Use `./refstack-client test --help` for the full list of arguments.

6. Upload your results

    If you previously ran a test with refstack-client without the `--upload`
    option, you can upload your results to a RefStack API server by using the
    following command::
    
        ./refstack-client upload <Path of results file>
    
    The results file is a JSON file generated by refstack-client when a test has
    completed. This is saved in .tempest/.testrepository. When you use the
    `upload` command, you can also override the RefStack API server uploaded to
    with the `--url` option.
    
    Alternatively, you can use the `upload-subunit` command to upload results
    using an existing subunit file. This requires that you pass in the Keystone
    endpoint URL for the cloud that was tested to generate the subunit data

7. List uploaded test set
    You can list previously uploaded data from a RefStack API server by using
    the following command:

        ./refstack-client list --url <URL of the RefStack API server>


__Example of successfull verification(2016.01 - Liberty)__:

* [Result of verification OpenStack Liberty - 97/97 tests(required)](https://refstack.openstack.org/#/results/2949be81-f337-4c89-83eb-c0422ff5dac9)
* [Result of verification OpenStack Liberty - 117/128 tests(required, aliases, flaged)](https://refstack.openstack.org/#/results/b0956ceb-61d5-4df1-ac38-1a97623eae36)
* [Result of verification OpenStack Liberty - 117/331 tests(required, aliases, flaged, advisory)](https://refstack.openstack.org/#/results/53885123-43f1-4498-9d0a-ef8461ede103)


## Hacks

For reaching good result you should (only for config generated rally):

  * import 2nd Cirros image to OpenStack  
  
  * change tempest.conf in section [compute]
  
        image_ref = 68f2e450-c9d8-4937-b24b-96d0ca627967
        image_ref_alt = a4c37608-9cbd-4f31-9b2e-d1d328671f0e
        flavor_ref = 57e9e361-c846-4676-8626-72f6355bb3f5
        flavor_ref_alt = 1
    
   image_ref and image_ref_alt - 2 copies of Cirros image (should have different id)       
   flavor_ref and flavor_ref_alt - 2 different flavors

---
