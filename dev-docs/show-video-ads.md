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

Other than using a slightly different ad unit in your Prebid code on the page, you shouldn't have to make any major engineering changes from a standard Prebid setup.

There should be no changes required on the ad ops side, since the video units use the standard Prebid creative, line item targeting setup, etc.

Note that you have the option to show outstream ads with or without an ad server.  For more information, see the instructions below.

* TOC
{:toc }

## Prerequisites

+ Demand from a bidder adapter that supports the one of the following media types (see below for more information):
  - `video: { context: "outstream" }`
  - `video: { context: "instream" }`
+ If the unit is outstream, the bidder adapter must return a renderer for outstream video in its bid response (although the publisher can [optionally define another renderer](#publisher-defined-renderer)).

{: .alert.alert-success :}
**Further Reading on Renderers and Video Streams**  
For more information about video stream contexts, see [the pull request adding the video stream context](https://github.com/prebid/Prebid.js/pull/1483).  
For more technical information about renderers, see [the pull request adding the 'Renderer' type](https://github.com/prebid/Prebid.js/pull/1082)

## Option 1. With an ad server

This section will take you through the code you need to write to show video ads using Prebid.js.

### Step 1. Set up outstream video slot sizes
{: .no_toc}

In your standard Prebid preamble in the header, configure slot sizes to suit your page layout and/or the available demand for those sizes.

{% highlight js %}

var pbjs = pbjs || {};

// ...

var rightSlotSizes = [[ 300, 250 ], [ 300, 600 ], [ 300, 250 ], [ 100, 100 ]];

// ...

{% endhighlight %}

### Step 2. Set up your ad units with the desired video context
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

This example assumes you are using DFP as your ad server.

In order to render the ads, you will need a module that can build the correct VAST video URL for your ad server.  If you use DFP, you can use the [DFP Ad Server Video module](https://github.com/prebid/Prebid.js/blob/master/modules/dfpAdServerVideo.js). (See the module source code for its documentation.)

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
2. Construct the VAST XML parameters yourself

#### Prebid Cache and Video Ads

By default, Prebid.js (specifically the [DFP Ad Server Video module](https://github.com/prebid/Prebid.js/blob/master/modules/dfpAdServerVideo.js)) caches the ad unit's VAST XML, builds the request properly using the [DFP API](https://support.google.com/dfp_premium/answer/1068325?hl=en), and then returns a single VAST URL to the page to make it convenient to show video ads.

However, there may be reasons you want to turn off the cache.

(<span style="color: rgb(255,0,0);">FIXME</span>: <strong>such as?</strong>)

If you need to turn off the cache, you can do so using the [config API](http://prebid.org/dev-docs/publisher-api-reference.html#module_pbjs.setConfig):

`pbjs.setConfig({ usePrebidCache: false });`

(<span style="color: rgb(255,0,0);">FIXME</span>: <strong>does this default to true or false?</strong>)

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

## Option 2. Without an Ad Server

You can show outstream video ads directly from a demand partner without going through an ad server.

In this section we'll explain the setup at a high level. For a live example showing all the details, see [Outstream without an Ad Server](http://acdn.adnxs.com/prebid/demos/outstream-without-adserver/).

### Step 1. Set up ad units with the desired video context
{: .no_toc}

Set up your video ad units with the desired video context as shown below.  Note that the `code` in your video ad unit must match a div ID in your page's body where the ad will be inserted.

As far as what fields are supported in the `bid.params.video` object, that will depend on the rendering options supported by your preferred bidder adaptor(s).  For more information, see [Bidders' Params]({{site.baseurl}}/dev-docs/bidders.html).

#### Instream

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

#### Outstream

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

## Option 3. With a Publisher-defined Renderer (Outstream only)

An outstream video bid response is accompanied by a video renderer in the form of a URL that points to a script containing the renderer software.  This software is used by Prebid.js to load and play the winning outstream bid.

If you would prefer to use a different renderer than the one provided in the bid response, you can specify that renderer on a per-outstream-ad-unit basis.  This renderer will be used to load and play the outstream ad, instead of the renderer that is supplied with the bid.  If both a publisher-supplied ad unit renderer and a bid renderer are present, the publisher-supplied renderer will be used.

To use a publisher-defined renderer, add a `renderer` property to your outstream ad unit as shown below.  Other than the ad unit, the rest of the setup will be the same as for [Option 1. With an ad server](#option-1-with-an-ad-server).

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

## Working Examples

Below, find links to end-to-end "working examples" demonstrating Prebid Video:

+ [AppNexus vs. Unruly](http://acdn.adnxs.com/prebid/alpha/unrulydemo.html) (Outstream)
+ [Outstream without an Ad Server](http://acdn.adnxs.com/prebid/demos/outstream-without-adserver/)

## Related Topics

+ [Outstream Video Example]({{site.baseurl}}/dev-docs/examples/outstream-video-example.html)

</div>
