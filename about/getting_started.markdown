---
layout: default
title:  Getting Started
---

## Prerequisites

Fog recommends using MRI 1.9.3 or 2.0.0.  MRI 1.8.7 and 1.9.2 are still supported by the Fog community, but are no longer supported by the Ruby community at large.  While not officially supported, Fog has been known to work with Ruby Enterprise Edition, Rubinus and JRuby.

## Installation

    $ gem install fog

## Credentials
Fog will continue to search for credentials in the following order until found:

1. service factory method (```Fog::Compute.new :provider => 'Rackspace', :rackspace_username => USERNAME, :rackspace_api_key => API_KEY```)
2. credential file specified by the environment variable ```FOG_RC```
3. ````.fog```` file in the user's home directory

*Note: When running fog's test suite, shindo will look for a .fog file in the tests directory*


This is an example of a ```.fog``` file:
 
	default:
	    rackspace_username: RACKSPACE_USERNAME
	    rackspace_api_key: RACKSPACE_API_KEY
	    public_key_path: ~/.ssh/fog_rsa.pub
	    private_key_path: ~/.ssh/fog_rsa
	
	provider2:
		provider_username: USERNAME
		provider_api_key: API_KEY

The ```.fog``` file is in YAML format. The top level level keys define the group of credentials. The nested key-value pairs define the credentials used by Fog. By default Fog will use the ```default``` group. This value can be overridden by defining the environment variable ```FOG_CREDENTIAL```.

Valid keys are as follows:

<table>
	<tr>
		<th>Key</th><th>Description</th>
	</tr>
    <tr><td>aws_access_key_id</td><td></td></tr>
    <tr><td>aws_secret_access_key</td><td></td></tr>
    <tr><td>bare_metal_cloud_password</td><td></td></tr>
    <tr><td>bare_metal_cloud_username</td><td></td></tr>
    <tr><td>bluebox_api_key</td><td></td></tr>
    <tr><td>bluebox_customer_id</td><td></td></tr>
    <tr><td>brightbox_client_id</td><td></td></tr>
    <tr><td>brightbox_secret</td><td></td></tr>
    <tr><td>clodo_api_key</td><td></td></tr>
    <tr><td>clodo_username</td><td></td></tr>
    <tr><td>cloudstack_api_key</td><td></td></tr>
    <tr><td>cloudstack_host</td><td></td></tr>
    <tr><td>cloudstack_secret_access_key</td><td></td></tr>
    <tr><td>dnsimple_email</td><td></td></tr>
    <tr><td>dnsimple_password</td><td></td></tr>
    <tr><td>dnsmadeeasy_api_key</td><td></td></tr>
    <tr><td>dnsmadeeasy_secret_key</td><td></td></tr>
    <tr><td>go_grid_api_key</td><td></td></tr>
    <tr><td>go_grid_shared_secret</td><td></td></tr>
    <tr><td>google_storage_access_key_id</td><td></td></tr>
    <tr><td>google_storage_secret_access_key</td><td></td></tr>
    <tr><td>hp_account_id</td><td></td></tr>
    <tr><td>hp_secret_key</td><td></td></tr>
    <tr><td>hp_tenant_id</td><td></td></tr>
    <tr><td>ibm_password</td><td></td></tr>
    <tr><td>ibm_username</td><td></td></tr>
    <tr><td>libvirt_ip_command</td><td></td></tr>
    <tr><td>libvirt_password</td><td></td></tr>
    <tr><td>libvirt_uri</td><td></td></tr>
    <tr><td>libvirt_username</td><td></td></tr>
    <tr><td>linode_api_key</td><td></td></tr>
    <tr><td>local_root</td><td></td></tr>
    <tr><td>openstack_api_key</td><td></td></tr>
    <tr><td>openstack_auth_url</td><td></td></tr>
    <tr><td>openstack_region</td><td></td></tr>
    <tr><td>openstack_tenant</td><td></td></tr>
    <tr><td>openstack_username</td><td></td></tr>
    <tr><td>ovirt_password</td><td></td></tr>
    <tr><td>ovirt_url</td><td></td></tr>
    <tr><td>ovirt_username</td><td></td></tr>
    <tr><td>private_key_path</td><td></td></tr>
    <tr><td>public_key_path</td><td></td></tr>
    <tr><td>rackspace_api_key</td><td>Rackspace API key</td></tr>
    <tr><td>rackspace_cdn_ssl</td><td></td></tr>
    <tr><td>rackspace_servicenet</td><td></td></tr>
    <tr><td>rackspace_username</td><td> Rackpace username</td></tr>
    <tr><td>stormondemand_password</td><td></td></tr>
    <tr><td>stormondemand_username</td><td></td></tr>
    <tr><td>terremark_password</td><td></td></tr>
    <tr><td>terremark_username</td><td></td></tr>
    <tr><td>voxel_api_key</td><td></td></tr>
    <tr><td>vsphere_password</td><td></td></tr>
    <tr><td>vsphere_username</td><td></td></tr>
    <tr><td>zerigo_email</td><td></td></tr>
    <tr><td>zerigo_token</td><td></td></tr>
</table>

## Debugging
You can turn on debug logging by setting the ```DEBUG``` environment variable. You can turn on request logging by setting the ```EXCON_DEBUG``` environment variable.

## Setting Up Local Storage

We will be using local storage in the example.  Local storage provides the same api that cloud storage services in fog do, but without the bother of needing to signup for stuff or pay extra money.

First, make a local directory to hold your data.

    $ mkdir ~/fog

Now we can start writing our script, first off we should require fog.

    require 'rubygems'
    require 'fog'

Now in order to play with our data we need to setup a storage connection.

    storage = Fog::Storage.new({
      :local_root => '~/fog',
      :provider   => 'Local'
    })

`storage` will now contain our storage object, configured to use the Local provider from our specified directory.

## Storing Data

Now that you have cleared the preliminaries you are ready to start storing data. Storage providers in fog segregate files into `directories` to make it easier to organize things. So lets create a directory so we can see that in action.

    directory = storage.directories.create(
      :key => 'data'
    )

To make sure it was created you can always check in your filesystem, but we can also check from inside fog.

    storage.directories

Progress! Now it is time to actually create a file inside our new directory.

    file = directory.files.create(
      :body => 'Hello World!',
      :key  => 'hello_world.txt'
    )

We should now have our file, first we can open it up and make sure we are on the right track.

    $ open ~/fog/hello_world.txt

It is much more likely that you will want to see what files you have from inside fog though.

    directory.files

Now that we have run through all the basics, lets clean up our mess.

    file.destroy
    directory.destroy

After that you should be able to check your directory list in fog or your filesystem and see you are safely back to square one.

## Next Steps

Using the same interface you can also practice working against a real provider (such as Amazon S3). Rather than worrying about signing up for an account right away though, we can use mocks to simulate S3 while we practice.

This time we will turn on mocking and then, just like before, we will need to make a connection.

    Fog.mock!
    storage = Fog::Storage.new({
      :aws_access_key_id      => 'fake_access_key_id',
      :aws_secret_access_key  => 'fake_secret_access_key',
      :provider               => 'AWS'
    })

You may notice that we used bogus credentials, this is fine since we are just simulating things. To use real S3 you can simply omit Fog.mock! and swap in your real credentials.

If you'd like to turn off mocking after turning it on, you can do it at any time and every subsequent connection will be a real connection.

    # Turn on mocking
    Fog.mock!

    # Create a mock connection to S3
    storage = Fog::Storage.new({
      :aws_access_key_id => "asdf",
      :aws_secret_access_key => "asdf",
      :provider => "AWS"
    })

    # Turn off mocking
    Fog.unmock!

    # Create a real connection to S3
    storage = Fog::Storage.new({
      :aws_access_key_id => "asdf",
      :aws_secret_access_key => "asdf",
      :provider => "AWS"
    })

Don't worry about your losing mock data, it stays around until you reset it or until your process exits.

    # Reset all mock data
    Fog::Mock.reset

Congratulations and welcome to the cloud!  Continue your journey at [fog.io](http://fog.io)
