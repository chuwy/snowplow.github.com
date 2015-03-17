---
layout: post
shortenedlink: Snowplow 62 released
title: Snowplow 62 Tropical Parula released
tags: [snowplow, beanstalk, clojure collector, emretlrunner]
author: Alex
category: Releases
---

We are pleased to announce the immediate availability of Snowplow 62, Tropical Parula. This release is designed to fix an incompatibility issue between r61's EmrEtlRunner and some older Elastic Beanstalk configurations. It also includes some other EmrEtlRunner improvements.

![tropical-parulas] [tropical-parulas]

Many thanks to Snowplow community member [Dani Solà] [danisola] from Simply Business for his contribution to this release!

1. [Fix to support legacy Beanstalk access logs](/blog/2015/05/17/snowplow-r62-tropical-parula-released/#emretlrunner-fix)
2. [Custom bootstrap actions](/blog/2015/05/17/snowplow-r62-tropical-parula-released/#bootstrap-actions)
3. [Other improvements to EmrEtlRunner](/blog/2015/05/17/snowplow-r62-tropical-parula-released/#emretlrunner-improvements)
3. [Upgrading](/blog/2015/05/17/snowplow-r62-tropical-parula-released/#upgrading)
4. [Getting help](/blog/2015/05/17/snowplow-r62-tropical-parula-released/#help)

<!--more-->

<h2><a name="emretlrunner-fix">1. Fix to support legacy Beanstalk access logs</a></h2>

After the [release of r61 Pygmy Parrot] [r61-release], we became aware that the updated file handling code for access logs generated by the Clojure Collector did not work with certain legacy Elastic Beanstalk environments, thus:

* Middle portion of access log filename is `tomcat7_rotated` - works fine
* Middle portion of access log filename is `tomcat8_rotated` - works fine
* Middle portion of access log filename is `tomcat7` - EmrEtlRunner does **not** move logs to Staging bucket

This is an easy issue to diagnose: if it affects you, then following an upgrade to r61 Pygmy Parrot, your Snowplow pipeline will copy **no** Clojure Collector access logs to Staging, and thus generate **no** enriched events.

This issue ([#1480] [issue-1480]) is resolved in this release: EmrEtlRunner now supports all Clojure Collector access log filename formats again.

<h2><a name="bootstrap actions">2. Custom bootstrap actions</a></h2>

The EmrEtlRunner now has support for adding one or more of your own custom bootstrap actions ([#1405] [issue-1405]). This is particularly useful if you are running your own Hadoop job steps as part of your scheduled jobflow on EMR. Many thanks to [Dani Solà] [danisola] for contributing this feature.

You simply set your custom bootstrap actions in your EmrEtlRunner's `config.yml` as an array:

{% highlight yaml %}
:emr:
  ...
  :ec2_key_name: ADD HERE
  :bootstrap:
    - s3://mybucket1/filename1
    - s3://mybucket2/filename2
  :software:
    ...
{% endhighlight %}

<h2><a name="emretlrunner-improvements">3. EmrEtlRunner improvements</a></h2>

We have made a variety of improvements "under the hood" to EmrEtlRunner:

* EmrEtlRunner now tolerates more exception types in EmrJob's wait_for ([#358] [issue-358]). This should reduce the incidence of monitoring failures during EMR runs
* We have bumped the version of Contracts to 0.7 (#1498), and moved `include Contracts` into classes and modules following best practice ([#1438] [issue-1438])
* The missing `:archive:` property has been added into the `BucketHash` ([#1475] [issue-1475])
* We have removed `time_diff` as a dependency because it was no longer used ([#1352] [issue-1352])
* The breaking test in the EmrEtlRunner's test suite is now fixed ([#1287] [issue-1287]). The test suite now passes again

<h2><a name="upgrading">4. Upgrading</a></h2>

<div class="html">
<h3><a name="upgrading-emretlrunner">4.1 Upgrading your EmrEtlRunner</a></h3>
</div>

You need to update EmrEtlRunner to the latest code (**0.13.0**) on GitHub:

{% highlight bash %}
$ git clone git://github.com/snowplow/snowplow.git
$ git checkout r62-tropical-parula
$ cd snowplow/3-enrich/emr-etl-runner
$ bundle install --deployment
$ cd ../../4-storage/storage-loader
$ bundle install --deployment
{% endhighlight %}

You **must** also update your EmrEtlRunner's configuration file, or else you will get a Contract failure on start. See the next section for details.

<div class="html">
<h3><a name="configuring-emretlrunner">4.2 Updating your EmrEtlRunner's configuration</a></h3>
</div>

Whether or not you use the new bootstrap option, you must update your EmrEtlRunner's `config.yml` file to include an entry for it:

In the `:emr:` section of your EmrEtlRunner's `config.yml` file, add in a `:bootstrap:` property like so:

{% highlight yaml %}
:emr:
  ...
  :ec2_key_name: ADD HERE
  :bootstrap: []          # No custom boostrap actions
  :software:
    ...
{% endhighlight %}

For a complete example, see our [sample `config.yml` template] [emretlrunner-config-yml].

<h2><a name="help">5. Getting help</a></h2>

For more details on this release, please check out the [r62 Tropical Parula Release Notes] [r62-release] on GitHub. 

If you have any questions or run into any problems, please [raise an issue] [issues] or get in touch with us through [the usual channels] [talk-to-us].

[tropical-parulas]: /assets/img/blog/2015/03/tropical-parulas.jpg

[danisola]: https://github.com/danisola

[issue-358]: https://github.com/snowplow/snowplow/issues/358
[issue-1287]: https://github.com/snowplow/snowplow/issues/1287
[issue-1352]: https://github.com/snowplow/snowplow/issues/1352
[issue-1405]: https://github.com/snowplow/snowplow/issues/1405
[issue-1475]: https://github.com/snowplow/snowplow/issues/1475
[issue-1480]: https://github.com/snowplow/snowplow/issues/1480

[emretlrunner-config-yml]: https://github.com/snowplow/snowplow/blob/master/3-enrich/emr-etl-runner/config/config.yml.sample

[r61-release]: /blog/2015/03/02/snowplow-r61-pygmy-parrot-released
[r62-release]: https://github.com/snowplow/snowplow/releases/tag/r62-tropical-parula
[issues]: https://github.com/snowplow/snowplow/issues
[talk-to-us]: https://github.com/snowplow/snowplow/wiki/Talk-to-us