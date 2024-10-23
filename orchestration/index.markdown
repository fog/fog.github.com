---
layout: default
title:  Orchestration
---

Many cloud providers offer APIs for managing the creation and management of multiple resources that
maybe related and/or dependent upon each other. The Orchestration model within Fog aims to provide
a common API for these orchestration style service APIs.

## Setup

First, make sure you have fog installed:

    gem install fog

For this example we will use the AWS CloudFormation service. Using your credentials, initialize a
connection to the service:

    require 'rubygems'
    require 'fog'

    # create connection to the service
    orchestration = Fog::Orchestration.new(
      :provider => :aws,
      :aws_access_key_id => AWS_ACCESS_KEY_ID,
      :aws_secret_access_key => AWS_SECRET_ACCESS_KEY,
      :region => 'us-east-1'
    )

## Creating a new stack

Now that a connection has been established, lets create a simple stack. First, lets define our stack within
a JSON template:

    # my_stack.json
    {
      "AWSTemplateFormatVersion": "2010-09-09",
      "Description": "My Stack",
      "Parameters": {
        "ImageId": {
          "Description": "Image ID for compute instance",
          "Type": "String",
          "Default": "ami-6aad335a"
        },
        "InstanceSize": {
          "Description": "Size of compute instance",
          "Type": "String",
          "Default": "m1.small"
        }
      },
      "Resources": {
        "MyEc2Instance": {
          "Type": "AWS::EC2::Instance",
          "Properties": {
            "ImageId": {
              "Ref": "ImageId"
            },
            "InstanceType": {
              "Ref": "InstanceSize"
            }
          }
        }
      },
      "Outputs": {
        "MyInstanceId": {
          "Description": "Resource ID",
          "Value": {
            "Ref": "MyInstance"
          }
        },
        "MyInstanceIp": {
          "Description": "Resource IP",
          "Value": {
            "Fn::GetAtt": [
              "MyInstance",
              "PublicIp"
            ]
          }
        }
      }
    }

Next we use this template to create the stack using the orchestration connection we defined above:

    stack = orchestration.stacks.new(
      :stack_name => 'my_stack',
      :template_description => 'My Template',
      :template => File.read('my_stack.json')
    )
    stack.save

## Stack information

Now that a stack exists, we can ask for details about the stack

### Stack resources

A stack consists of a collection of resources. The stack contains a list of these resources
for inspection:

    stack.resources.each do |resource|
      puts "Resource Information: #{resource.logical_resource_id}"
      puts "  * Type:            #{resource.resource_type}"
      puts "  * Physical ID:     #{resource.physical_resource_id}"
      puts "  * Resource Status: #{resource.resource_status}"
      puts "  * Updated:         #{resource.updated_time}"
    end

### Stack events

A stack contains a series of events. These events are a history of the state changes the
resources within the stack have undergone during the stack's lifetime. Listing the stack
events:

    stack.events.each do |event|
      puts "Event Information: #{event.logical_resource_id}"
      puts "  * Physical ID:   #{event.physical_resource_id}"
      puts "  * Status:        #{event.resource_status}"
      puts "  * Status Reason: #{event.resource_status_reason}"
      puts "  * Event Time:    #{event.event_time}"
    end

### Stack outputs

The outputs defined by the stack template can also be accessed:

    stack.outputs.each do |output|
      puts "#{output.key} = #{output.value}"
    end

### Template related information

The serialized representation of the template can be accessed:

    puts stack.template

These templates are generally a JSON blob and can easily be loaded:

    puts Fog::JSON.decode(stacks.first.template)

The parameters used to create the stack:

    stack.parameters.each do |key, value|
      puts "#{key} = #{value}"
    end

Capabilities enabled for the stack:

    stack.capabilities.each do |capability|
      puts capability
    end

Rollback is disabled on error:

    puts "Rollback is: #{stack.disable_rollback ? 'disabled' : 'enabled'}"

### Other stack information

The stack has other information available as well:

    # name of stack
    stack.stack_name
    # ID of stack
    stack.id
    # description of template
    stack.description_template
    # stack creation time
    stack.creation_time
    # current status of stack
    stack.stack_status

## Update a stack

Stacks can be updated, changing parameters or the underlying template
describing the stack. First lets update the stack template, adding a
new instance to the stack:

    # my_updated_stack.json
    {
      "AWSTemplateFormatVersion": "2010-09-09",
      "Description": "My Stack",
      "Parameters": {
        "ImageId": {
          "Description": "Image ID for compute instance",
          "Type": "String",
          "Default": "ami-6aad335a"
        },
        "InstanceSize": {
          "Description": "Size of compute instance",
          "Type": "String",
          "default": "m1.small"
        }
      },
      "Resources": {
        "MyEc2Instance": {
          "Type": "AWS::EC2::Instance",
          "Properties": {
            "ImageId": {
              "Ref": "ImageId"
            },
            "InstanceType": {
              "Ref": "InstanceSize"
            }
          }
        },
        "MyNewInstance": {
          "Type": "AWS::EC2::Instance",
          "Properties": {
            "ImageId": {
              "Ref": "ImageId"
            },
            "InstanceType": {
              "Ref": "InstanceSize"
            }
          }
        }
      },
      "Outputs": {
        "MyInstanceId": {
          "Description": "Resource ID",
          "Value": {
            "Ref": "MyInstance"
          }
        },
        "MyInstanceIp": {
          "Description": "Resource IP",
          "Value": {
            "Fn::GetAtt": [
              "MyInstance",
              "PublicIp"
            ]
          }
        }
      }
    }

Now, update the size of the instances and apply the updated template:

    stack.parameters['InstanceSize'] = 'm1.medium'
    stack.template = File.read('my_updated_stack.json')
    stack.save

## Delete a stack

Stacks can be destroyed:

    stack.destroy

## Build stacks for better management

Grouping related resources into orchestration style stacks allows for easier
management and usage of resources in the cloud. Using the orchestration
models provide a consistent API for utilizing orchestration services available
from cloud providers.