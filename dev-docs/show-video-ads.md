---
layout: page
title: Show Video Ads
description: Show Video Ads with Prebid.js
pid: 10
top_nav_section: dev_docs
nav_section: prebid-video
---

<div class="bs-docs-section" markdown="1">

# Show Video Ads
{: .no_toc}

This page has information you'll need to set up Prebid.js to show video ads.

This includes the following types of video setups:

+ Outstream video with an ad server
+ Outstream video *without* an ad server
+ Outstream with a publisher-defined renderer
+ Instream video (with an ad server)

See below for instructions for showing each type of ad.

Other than using a slightly different ad unit in your Prebid code on the page, you shouldn't have to make any major engineering changes from a standard Prebid setup to show video ads.

There should be no changes required [on the ad ops side]({{site.baseurl}}/adops/setting-up-prebid-video-in-dfp.html), since the video units use the standard Prebid creative, line item targeting setup, etc.

* TOC
{:toc }

## Overview

As of [version 0.29](https://github.com/prebid/Prebid.js/releases/tag/0.29.0), video-enabled bidders can participate on both instream and outstream impressions.

This support is enabled by the ad unit including a `mediaTypes` parameter, which can be used to configure video ad units to be in one of the following *video stream contexts*:

+ Instream
+ Outstream

No changes are required to bidder adapters that already support the `video` media type, although the `mediaTypes` parameter (which includes the video context) is now passed to adapters in the `callBids` parameter.  Adapters can use this to target instream and outstream impressions differently if they wish.

{: .alert.alert-success :}
For more information about video stream contexts, see [the pull request adding the `mediaTypes` parameter](https://github.com/prebid/Prebid.js/pull/1483).

## Prerequisites

+ Demand from a bidder adapter that supports the `video` media type.
+ If the unit is outstream, the bidder adapter must return a renderer for outstream video in its bid response (although the publisher can [optionally define another renderer](#publisher-defined-renderer)).

{: .alert.alert-success :}
For more information about video renderers, see [the pull request adding the 'Renderer' type](https://github.com/prebid/Prebid.js/pull/1082)

## Outstream with an ad server

### Step 1. Set up outstream video slot sizes
{: .no_toc}

In your standard Prebid preamble in the header, configure slot sizes to suit your page layout and/or the available demand for those sizes.

{% highlight js %}

var pbjs = pbjs || {};

// ...

var rightSlotSizes = [[ 300, 250 ], [ 300, 600 ], [ 300, 250 ], [ 100, 100 ]];

// ...

{% endhighlight %}

### Step 2. Set up your ad units
{: .no_toc}

Still in the header, set up your ad units with the desired video context.   Note that the `code` in your video ad unit must match a div ID in your page's body where the ad will be inserted.

As far as what fields are supported in the `bid.params.video` object, that will depend on the rendering options supported by your preferred bidder adaptor(s).  For more information, see [Bidders' Params]({{site.baseurl}}/dev-docs/bidders.html).

{% highlight js %}

var videoAdUnits = [
  {
    code: 'video1',
    sizes: [ 640, 480 ],
    mediaTypes: {
        video: {
            context: 'outstream'
        }
    },
    bids: [
      {
        bidder: 'appnexusAst',
        params: {
          placementId: '5768085',
          video: {
            skippable: true,
            playback_method: [ 'auto_play_sound_off' ]
          }
        }
      }
    ]
  }
];

{% endhighlight %}

### Step 3. Request bids, render the ad
{: .no_toc}

{: .alert.alert-success :}
This example assumes you are using DFP as your ad server.

In order to render the ads, you will need a module that can build the correct VAST video URL for your ad server.  If you use DFP, you can use the [DFP Ad Server Video module](https://github.com/prebid/Prebid.js/blob/master/modules/dfpAdServerVideo.js). (See the module source code for its API documentation.)

{% highlight js %}
pbjs.que.push(function() {
    pbjs.addAdUnits(videoAdUnits);
    pbjs.requestBids({
        bidsBackHandler: function(bids) {
            var videoUrl = pbjs.adServers.dfp.buildVideoUrl({
                adUnit: videoAdUnits[0],
                params: {
                    iu: '/19968336/prebid_video_adunit'
                }
            });
        }
    })
});
{% endhighlight %}

{: .alert.alert-info :}
**Non-DFP Ad Servers**  
If you use a different ad server than DFP, you can either:  
1. Get them to write a module providing support for Prebid.js  
2. Construct the VAST XML parameters yourself (see [the source code of the DFP ad server video module](https://github.com/prebid/Prebid.js/blob/master/modules/dfpAdServerVideo.js) for how)

{% include dev-docs/prebid-cache-explainer.md %}

### Step 4. Show ads on the page as normal
{: .no_toc}

In the body of the page, insert your ads as usual:

{% highlight html %}

<div id='video1'>
  <p>Prebid Outstream Video Ad</p>
  <script type='text/javascript'>
    googletag.cmd.push(function () {
      googletag.display('video2');
    });
  </script>
</div>

{% endhighlight %}

## Outstream without an Ad Server

You can show outstream video ads directly from a demand partner without going through an ad server.

In this section we'll explain the setup at a high level. For a live example showing all the details, see [Outstream without an Ad Server](http://acdn.adnxs.com/prebid/demos/outstream-without-adserver/).

### Step 1. Set up ad units
{: .no_toc}

Set up your video ad units with the desired video context as shown below.  Note that the `code` in your video ad unit must match a div ID in your page's body where the ad will be inserted.

As far as what fields are supported in the `bid.params.video` object, that will depend on the rendering options supported by your bidder adaptor(s).  For more information, see [Bidders' Params]({{site.baseurl}}/dev-docs/bidders.html).

{% highlight js %}
var videoAdUnits = [{
    code: 'video1',
    sizes: [640, 480],
    mediaTypes: {
        video: {
            context: 'outstream'
        }
    },

    /* AppNexus AST adapter contains a renderer so this isn't
    required, but if present it will be used in favor of the renderer
    returned with the bid response. */
    renderer: {
        url: 'http://cdn.adnxs.com/renderer/video/ANOutstreamVideo.js',
        render: function(bid) {
            ANOutstreamVideo.renderAd({
                targetId: bid.adUnitCode,
                adResponse: bid.adResponse
            });
        }
    },

    bids: [{
        bidder: 'appnexusAst',
        params: {
            placementId: '5768085'
            video: {
                skippable: true,
                playback_method: ['auto_play_sound_off']
            }
        }
    }]
}];
{% endhighlight %}

### Step 2. Request bids, and render the returned ad in a callback
{: .no_toc}

In the Prebid.js event queue, you'll need to add a function that:

1. Adds your video ad units
2. Requests bids, adding a callback that:
    1. Gets the targeting for your ad unit
    2. Renders the ad

{% highlight js %}
pbjs.que.push(function () {
    pbjs.addAdUnits(videoAdUnits);
    pbjs.requestBids({
        timeout: 3000,
        bidsBackHandler: function (bids) {
            /*
            `getAdServerTargetingForAdUnitCode` is used as an easy
            way to get the ad ID, it doesn't actually require you
            to use an ad server.
            */
            var video1 = pbjs.getAdserverTargetingForAdUnitCode('video1');
            pbjs.renderAd(document, video1.hb_adid);
        }
    });
});
{% endhighlight %}

For more information, see the API documentation for:

+ [requestBids](https://github.com/prebid/Prebid.js/blob/master/src/prebid.js#L351)
+ [getAdserverTargetingForAdUnitCode](https://github.com/prebid/Prebid.js/blob/master/src/prebid.js#L134)
+ [renderAd](https://github.com/prebid/Prebid.js/blob/master/src/prebid.js#L262)

<a name="publisher-defined-renderer"/>

## Outstream with a Publisher-defined Renderer

An outstream video bid response is accompanied by a video renderer in the form of a URL that points to a script containing the renderer software.  This software is used by Prebid.js to load and play the winning outstream bid.

If you would prefer to use a different renderer than the one provided in the bid response, you can specify that renderer on a per-outstream-ad-unit basis.  This renderer will be used to load and play the outstream ad instead of the renderer that is supplied with the bid.

To use a publisher-defined renderer, add a `renderer` property to your outstream ad unit as shown below.  Other than the ad unit, the rest of the setup will be the same as for [Option 1. With an ad server](#option-1-with-an-ad-server).

{: .alert.alert-info :}
If both a publisher-supplied ad unit renderer and a bid renderer are present, the publisher-supplied renderer will be used.

{% highlight js %}
var videoAdUnits = [{
    code: 'video1',
    sizes: [640, 480],
    mediaTypes: {
        video: {
            context: 'outstream'
        }
    },

    /* The adapter response (AppNexus AST in this case) contains a
    renderer so this isn't required, but if present it will be used in
    favor of the renderer returned with the bid response. */
    renderer: {
        url: 'http://cdn.adnxs.com/renderer/video/ANOutstreamVideo.js',
        render: function(bid) {
            ANOutstreamVideo.renderAd({
                targetId: bid.adUnitCode,
                adResponse: bid.adResponse
            });
        }
    },

    bids: [{
        bidder: 'appnexusAst',
        params: {
            placementId: '5768085'
            video: {
                skippable: true,
                playback_method: ['auto_play_sound_off']
            }
        }
    }]
}];
{% endhighlight %}

## Instream with an ad server

### Step 1. Set up video slot sizes
{: .no_toc}

In your standard Prebid preamble in the header, configure slot sizes to suit your page layout and/or the available demand for those sizes.

{% highlight js %}

var pbjs = pbjs || {};

// ...

var rightSlotSizes = [[ 300, 250 ], [ 300, 600 ], [ 300, 250 ], [ 100, 100 ]];

// ...

{% endhighlight %}

### Step 2. Set up your ad units
{: .no_toc}

Still in the header, set up your ad units with the desired video context.   Note that the `code` in your video ad unit must match a div ID in your page's body where the ad will be inserted.

As far as what fields are supported in the `bid.params.video` object, that will depend on the rendering options supported by your preferred bidder adaptor(s).  For more information, see [Bidders' Params]({{site.baseurl}}/dev-docs/bidders.html).

{% highlight js %}
var videoAdUnits = [{
    code: 'video1',
    sizes: [640, 480],
    mediaTypes: {
        video: {
            context: 'instream'
        }
    },
    bids: [{
        bidder: 'appnexusAst',
        params: {
            placementId: '9333431'
        }
    }]
}];
{% endhighlight %}

### Step 3. Request bids, render the ad
{: .no_toc}

<span style="color: rgb(255,0,0);">FIXME</span>: <strong>What is the right way to set up instream video?  How does it differ from what's shown below?  As of now I'm much more familiar with the outstream method and examples.</strong>

{: .alert.alert-success :}
This example assumes you are using DFP as your ad server.

In order to render the ads, you will need a module that can build the correct VAST video URL for your ad server.  If you use DFP, you can use the [DFP Ad Server Video module](https://github.com/prebid/Prebid.js/blob/master/modules/dfpAdServerVideo.js) as shown in the code sample below to correctly build the VAST video URL.

{% highlight js %}
pbjs.que.push(function() {
    pbjs.addAdUnits(videoAdUnits);
    pbjs.requestBids({
        bidsBackHandler: function(bids) {
            var videoUrl = pbjs.adServers.dfp.buildVideoUrl({
                adUnit: videoAdUnits[0],
                params: {
                    iu: '/19968336/prebid_video_adunit'
                }
            });
        }
    })
});
{% endhighlight %}

{: .alert.alert-info :}
**Non-DFP Ad Servers**  
If you use a different ad server than DFP, you can either:  
1. Get them to write a module providing support for Prebid.js  
2. Construct the VAST XML to your ad server's requirements

{% include dev-docs/prebid-cache-explainer.md %}

### Step 4. Show ads on the page as normal
{: .no_toc}

In your video player of choice, insert instream ads as normal.

<span style="color: rgb(255,0,0);">FIXME</span>: <strong>Pretty sure this snippet is now how to show instream ads in a video player.  Ask if you can get an example.</strong>

{% highlight html %}

<div id='video1'>
  <p>Prebid Outstream Video Ad</p>
  <script type='text/javascript'>
    googletag.cmd.push(function () {
      googletag.display('video2');
    });
  </script>
</div>

{% endhighlight %}

## Working Examples

Below, find links to end-to-end "working examples" demonstrating Prebid Video.

+ [AppNexus vs. Unruly](http://acdn.adnxs.com/prebid/alpha/unrulydemo.html) (Outstream)
+ [Outstream without an Ad Server](http://acdn.adnxs.com/prebid/demos/outstream-without-adserver/)
+ [INSTREAM EXAMPLE NEEDED]()

## Related Topics

+ [Outstream Video Example]({{site.baseurl}}/dev-docs/examples/outstream-video-example.html)

</div>
