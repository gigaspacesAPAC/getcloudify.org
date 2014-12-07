---
layout: bt_wiki
title: Policies Guide
category: Guides
publish: true
abstract: "A guide to Cloudify Policies"
pageord: 500

types_yaml_link: http://www.getcloudify.org/spec/cloudify/3.1/types.yaml
diamond_plugin_ref: plugin-diamond.html
---

{%summary%}This guide explains what policies are and how to use them{%endsummary%}


# Introduction to Policies

Policies provide a way of analyzing a stream of events that correspond to a group of nodes (and their instances).
The analysis process occurs in real time and enables triggering actions based on its outcome.

The stream of events for each node instance is read from a deployment specific RabbitMQ queue by the policy engine ([Riemann](http://riemann.io/)). It may come from any source but is typically generated by a monitoring agent installed the node's host. (See: [Diamond Plugin]({{page.diamond_plugin_ref}})).

The analysis is performed using [Riemann](http://riemann.io/). Cloudify starts a Riemann core for each deployment with its own Riemann configuration. The Riemann configuration is generated based on the blueprint's `groups` section that will be described later in this guide.

A part of the API that Cloudify provides on top of Riemann, allows activating policy triggers. Usually, that simply means executing a workflow in response to some state (e.g. The tomcat CPU usage was above 90% for the last five minutes).
That workflow may increase the number of node instances, for example, to handle an increased load.

The policies logic itself is written in [Clojure](http://clojure.org/) using Riemann's API. Cloudify adds a thin API layer on top of these API's.

# Using Policies

Policies are configured in the `groups` section of a blueprint.
An example:

{% highlight yaml %}
groups:
  # arbitrary group name
  my_group:

    # all nodes this group applies to
    # in this case, assume node_vm and mongo_vm
    # were defined in the node_templates section.
    # All events that belong to node instances of
    # these nodes will be processed by all policies specified
    # within the group
    members: [node_vm, mongo_vm]

    # each group specifies a set of policies
    policies:

      # arbitrary policy name
      my_policy:

        # the policy type to use. Here we use one
        # of the built-in policy types that identifies
        # host failures based on a keep alive mechanism
        type: cloudify.policies.types.host_failure

        # policy specific configuration
        properties:
          service: cpu

        # This section specifies what should be
        # triggered when the policy is "triggered"
        # (more than one trigger can be specified)
        triggers:
          # arbitrary trigger name
          execute_autoheal_workflow:

            # using the built-in 'execute_workflow' trigger
            type: cloudify.policies.triggers.execute_workflow

            # trigger specific configuration
            parameters:
              workflow: auto_heal
              workflow_parameters:
                # the failing node instance id is exposed by the
                # host_failure policy on the event that triggered the
                # execute workflow trigger. Access to properties on
                # that event are as demonstrated below
                node_id: { get_property: [SELF, node_id] }

{% endhighlight %}

# Built-in Policies

Cloudify comes with a number of built-in policies.

Built-in policies are declared in [`types.yaml`]({{page.types_yaml_link}}), which is usually imported either directly or indirectly via other imports.

{% highlight yaml %}
# snippet from types.yaml
policy_types: {}
    # TODO

{% endhighlight %}

Built-in policies are not special in any way - they use the same API any other custom policy is able to use.

# Writing a Custom Policy

Advanced users may wish to write custom policies.
To learn how to write a custom policy, refer to the [policies authoring guide](guide-authoring-policies.html).