#### Prebid Cache and Video Ads
{: .no_toc}

By default, Prebid.js (specifically the [DFP Ad Server Video module](https://github.com/prebid/Prebid.js/blob/master/modules/dfpAdServerVideo.js)) caches the ad unit's VAST XML, builds the request properly using the [DFP API](https://support.google.com/dfp_premium/answer/1068325?hl=en), and then returns a single VAST URL to the page to make it convenient to show video ads.

However, there may be reasons you want to turn off this cache.

(<span style="color: rgb(255,0,0);">FIXME</span>: <strong>Why would you want to do this?  I suspect control / reliability?</strong>)

If you need to turn off the cache, you can do so using the [config API](http://prebid.org/dev-docs/publisher-api-reference.html#module_pbjs.setConfig) like so:

`pbjs.setConfig({ usePrebidCache: false });`

(<span style="color: rgb(255,0,0);">FIXME</span>: <strong>Does this default to true or false?</strong>)
