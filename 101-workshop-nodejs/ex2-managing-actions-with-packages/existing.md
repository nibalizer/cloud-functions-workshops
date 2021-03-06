# Using Packages

## Browsing packages

IBM Cloud Functions comes pre-installed with a number of public packages, which include trigger feeds used to register triggers with event sources.

Actions in public packages can be used by anyone, the caller pays the invocation cost.

Using `ibmcloud fn` CLI you can get a list of packages in a namespace, list the entities in a package and get a description of the entities within a package.

Get a list of packages in the `/whisk.system` namespace.

```bash
ibmcloud fn package list /whisk.system
```

```text
packages
/whisk.system/alarms                      shared
/whisk.system/cloudant                    shared
/whisk.system/combinators                 shared
/whisk.system/cos                         shared
/whisk.system/github                      shared
/whisk.system/messaging                   shared
/whisk.system/pushnotifications           shared
/whisk.system/samples                     shared
/whisk.system/slack                       shared
/whisk.system/utils                       shared
/whisk.system/watson-speechToText         shared
/whisk.system/watson-textToSpeech         shared
/whisk.system/watson-translator           shared
/whisk.system/weather                     shared
/whisk.system/websocket                   shared
```

Get a list of entities in the `/whisk.system/cloudant` package.

```bash
ibmcloud fn package get --summary /whisk.system/cloudant
```

```bash
package /whisk.system/cloudant: Cloudant database service
   (parameters: *apihost, *bluemixServiceName, *dbname, *host, overwrite, *password, *username)
  action /whisk.system/cloudant/read: Read document from database
   (parameters: dbname, id, params)
  action /whisk.system/cloudant/write: Write document in database
   (parameters: dbname, doc)
   ...
  feed /whisk.system/cloudant/changes: Database change feed
   (parameters: dbname, filter, query_params)
```

This output shows that the Cloudant package provides many actions including `read` and `write`, and a trigger feed called `changes`. The `changes` feed causes triggers to be fired when documents are added to the specified Cloudant database.

The Cloudant package also defines the parameters `username`, `password`, `host`, and `dbname`. These parameters must be specified for the actions and feeds to be meaningful. The parameters allow the actions to operate on a specific Cloudant account.

{% hint style="info" %}
**Note:**
* Parameters listed under the package with a prefix `'*'` are **predefined, bound parameters**.
* Parameters without a `'*'` are those listed under the annotations for each entity.
* Furthermore, any parameters with the prefix `'**'` are **finalized bound parameters**. This means that they are immutable, and cannot be changed by the user.
{% endhint %}

## Viewing parameters

Any entity listed under a package inherits specific bound parameters from the package. To view the list of known parameters of an entity belonging to a package, you will need to run a `get --summary` of the individual entity.

Let's look more closely at the `read` action in the Cloudant package:

1. Get a description of the `/whisk.system/cloudant/read` action.

   ```bash
   ibmcloud fn action get --summary /whisk.system/cloudant/read
   ```

   ```text
   action /whisk.system/cloudant/read: Read document from database
      (parameters: *apihost, *bluemixServiceName, *dbname, *host, *id, params, *password, *username)
   ```

   This output shows that the Cloudant `read` action lists eight parameters, seven of which are predefined. These include the database and document ID (`id`) to retrieve.

## Invoking actions in a package

You can invoke actions in a package, just as with other actions. The next few steps show how to invoke the `greeting` action in the `/whisk.system/samples` package with different parameters.

1. Get a description of the `/whisk.system/samples/greeting` action.

   ```bash
   ibmcloud fn action get --summary /whisk.system/samples/greeting
   ```

   ```text
   action /whisk.system/samples/greeting: Returns a friendly greeting
      (parameters: name, place)
   ```

   Notice that the `greeting` action takes two parameters: `name` and `place`.

1. Invoke the action without any parameters.

   ```bash
   ibmcloud fn action invoke --result /whisk.system/samples/greeting
   ```

   ```text
   {
       "payload": "Hello, stranger from somewhere!"
   }
   ```

   The output is a generic message because no parameters were specified.

1. Invoke the action with parameters.

   ```bash
   ibmcloud fn action invoke --result /whisk.system/samples/greeting --param name Arya --param place Winterfell
   ```

   ```text
   {
       "payload": "Hello, Arya from Winterfell!"
   }
   ```

   Notice that the output uses the `name` and `place` parameters that were passed to the action.

## Creating and using package bindings

Although you can use the entities in a package directly, you might find yourself passing the same parameters to the action every time. You can avoid this by binding to a package and specifying default parameters. These parameters are inherited by the actions in the package.

For example, in the `/whisk.system/cloudant` package, you can set default `username`, `password`, and `dbname` values in a package binding and these values are automatically passed to any actions in the package.

In the following simple example, you bind to the `/whisk.system/samples` package.

1. Bind to the `/whisk.system/samples` package and set a default `place` parameter value.

   ```text
   ibmcloud fn package bind /whisk.system/samples valhallaSamples --param place Valhalla
   ```

   ```text
   ok: created binding valhallaSamples
   ```

1. Get a description of the package binding.

   ```text
   ibmcloud fn package get --summary valhallaSamples
   ```

   ```text
   package /namespace/valhallaSamples: Returns a result based on parameter place
      (parameters: *place)
    action /namespace/valhallaSamples/helloWorld: Demonstrates logging facilities
       (parameters: payload)
    action /namespace/valhallaSamples/greeting: Returns a friendly greeting
       (parameters: name, place)
    action /namespace/valhallaSamples/curl: Curl a host url
       (parameters: payload)
    action /namespace/valhallaSamples/wordCount: Count words in a string
       (parameters: payload)
   ```

   Notice that all the actions in the `/whisk.system/samples` package are available in the `valhallaSamples` package binding.

1. Invoke an action in the package binding

   ```text
   ibmcloud fn action invoke --result valhallaSamples/greeting --param name Odin
   ```

   ```text
   {
       "payload": "Hello, Odin from Valhalla!"
   }
   ```

   Notice from the result that the action inherits the `place` parameter you set when you created the `valhallaSamples` package binding.

1. Invoke an action and overwrite the default parameter value.

   ```text
   ibmcloud fn action invoke --result valhallaSamples/greeting --param name Odin --param place Asgard
   ```

   ```text
   {
       "payload": "Hello, Odin from Asgard!"
   }
   ```
{% hint style="info" %}
   Notice that the `place` parameter value that is specified with the action invocation overwrites the default value set in the `valhallaSamples` package binding.
{% endhint %}
