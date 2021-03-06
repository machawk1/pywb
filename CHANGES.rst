pywb 0.9.7 changelist
~~~~~~~~~~~~~~~~~~~~~

* wombat enchancements: support for mutation observers instead of ``setAttribute`` override with ``client.use_attr_observers`` setting.
  Can also disable worker override with ``skip_disable_worker``
  
* wombat fixes: Better check for self-redirect when proxying ``replace()`` and ``assign()``, use ``querySelectorAll()`` for dom selection

* wombat fixes: Don't remove trailing slash in ``extract_orig()``, treat slash and no-slash urls as distinct on the client (as expected).

* cdx-indexer: Validation of HTTP protocol and request verbs now optional. Any protocol and verb will be accepted, unless ``-v`` flag is used,
  allowing for indexing of content with custom verbs, unexpected protocol, etc...


pywb 0.9.6 changelist
~~~~~~~~~~~~~~~~~~~~~

* framed replay: fix bug where outer frame url was not updated (in inverse mode) after navigating inner frame.

* framed replay: lookup frame by id, ``replay_iframe``, instead of by using ``window.frames[0]`` to allow for more customization.

* fix typo in wombat ``no_rewrite_prefixes``


pywb 0.9.5 changelist
~~~~~~~~~~~~~~~~~~~~~

* s3 loading: support ``s3://`` scheme in block loader, allowing for loading index and archive files from s3. ``boto`` library must be installed seperately
  via ``pip install boto``. Attempt default boto auth path, and if that fails, attempt anonymous s3 connection.
  
* Wombat/Client-Side Rewrite Customizations: New ``rewrite_opts.client`` settings from ``config.yaml`` are passed directly to wombat as json. 
  
  Allows for customizing wombat as needed. Currently supported options are: ``no_rewrite_prefixes`` for ignoring rewrite
  on certain domains, and ``skip_dom``, ``skip_setAttribute`` and ``skip_postmessage`` options for disabling 
  those overrides. Example usage in config:
  
  ::

    rewrite_opts:
        ...
        client:
            no_rewrite_prefixes: ['http://dont-rewrite-this.example.com/']
  
            skip_setAttribute: true
            skip_dom: true
            skip_postmessage: true
  
  
* Revamp template setup: All templates now use shared env, which is created on first use or can be explicitly set (if embedding)
  via ``J2TemplateView.init_shared_env()`` call. Support for specifiying a base env, as well as custom template lookup paths also provided
  
* Template lookup paths can also be set via config options ``templates_dirs``. The default list is: ``templates``, ``.``, ``/`` in that order.

* Embedding improvements: move custom env (``REL_REQUEST_URI`` setup) into routers, should be able to call router created by ``create_wb_router()`` 
  directly with WSGI enviorn and receive a callable response.

* Embedding improvements: If set, the contents of ``environ['pywb.template_params']`` dictionary are added directly to Jinja context, allowing for custom template
  params to be passed to pywb jinja templates.

* Root collection support: Can specify a route with `''` which will be the root collection. Fix routing paths to ensure root collection is checked last.

* Customization: support custom route_class for cdx server and pass wbrequest to ``not_found_html``  error handlers.

* Manager: Validate collection names to start with word char and contain alphanum or dash only.

* CLI refactor: easier to create custom cli apps and pass params, inherit shared params. ``live-rewrite-server`` uses new system cli system,
  defaults to framed inverse mode. Also runs on ``/live/`` path by default. See ``live-rewrite-server -h`` for a list of current options.

* Add ``cookie_scope: removeall`` cookie rewriter, which will, remove all cookies from replay headers.

* Security: disable file:// altogether for live rewrite path.

* Fuzzy match: better support for custom replace string >1 character: leave string, and strip remainder before fuzzy query.

* Urlrewriter and wburl fixes for various corner cases.

* Rangecache: use url as key if digest not present.

* Framed replay: attempt to mitigate chrome OS X scrolling issue by disabling ``-webkit-transform: none`` in framed mode. 
  Improves scrolling on many pages but not always consistent (a chrome bug).


pywb 0.9.3 changelist
~~~~~~~~~~~~~~~~~~~~~

* framed replay mode: support ``framed_replay: inverse`` where the top frame is the canonical archival url and the inner frame has ``mp_`` modifier.

* wb.js: improved redirect check: only redirect to top frame in framed mode and compare decoded urls.

* charset detection: read first 1024 bytes to determine charset and add to ``Content-Type`` header if no charset is specified there.

* indexing: support indexing of WARC records with ``urn:`` values as target uris, such as those created by `wpull <https://github.com/chfoo/wpull>`_

* remove certauth module: now using standalone `certauth <http://github.com/ikreymer/certauth>`_ package.

* BlockLoader: use ``requests`` instead of ``urllib2``.

* cdx: %-encode any non-ascii chars found in cdx fields.

* cdx: showNumPages query always return valid result (not 404) for 0 pages. If <1 block, load cdx to determine if 1 page or none.


pywb 0.9.2 changelist
~~~~~~~~~~~~~~~~~~~~~

* Collections Manager: Allow adding any templates to shared directory, fix adding WARCs with relative path.

* Replay: Remove limit by HTTP ``Content-Length`` as it may be invalid (only using the record length).

* WARC Revisit-Resolution Improvements: Support indexes and warcs without any ``digest`` field. If no digest is found, attempt to look up
  the original WARC record from the ``WARC-Refers-To-Target-URI`` and ``WARC-Refers-To-Date`` only, even for same url revisits.
  (Previously, only used this lookup original url was different from revisit url)


pywb 0.9.1 changelist
~~~~~~~~~~~~~~~~~~~~~

* Implement pagination support for zipnum cluster and added to cdx server api:

  https://github.com/ikreymer/pywb/wiki/CDX-Server-API

* cdx server query: add support for ``url=*.host`` and ``url=host/*`` as shortcuts for ``matchType=domain`` and ``matchType=prefix``

* zipnum cdx cluster: support loading index shared from prefix path instead of seperate location file.

  The ``shard_index_loc`` config property may contain match and replace properties.
  Regex replacement is then used to obtain path prefix from the shard prefix path.

* wombat: fix `document.write()` rewriting to rewrite each element at a time and use underlying write for better compatibility.


pywb 0.9.0 changelist
~~~~~~~~~~~~~~~~~~~~~

* New directory-based configuration-less init system! ``config.yaml`` no longer required.

* New ``wb-manager`` collection manager for adding warcs, indexing, adding/removing templates, setting metadata.

  More details at: `Auto-Configuration and Wayback Collections Manager <https://github.com/ikreymer/pywb/wiki/Auto-Configuration-and-Wayback-Collections-Manager>`_

* Support for user metadata via per-collection ``metadata.yaml``

* Templates: improved/simpified home page and collection search page, show user metadata by default.

* Support for writing and reading new cdx JSON format (.cdxj), with searchable key followed by json dictionary: ``urlkey timestamp { ... }`` on each line

* ``cdx-indexer -j``: support for generating cdxj format

* ``cdx-indexer -mj``: support for minimal cdx format (in JSON format) only which skips reading the HTTP record.

    Fields included in minimal format are: urlkey, timestamp, original url, record length, digest, offset, and filename

* ``cdx-indexer --root-dir <dir>``: option for custom root dir for cdx filenames to be relative to this directory.

* ``wb-manager cdx-convert``: option to convert any existing cdx to new cdxj format, including ensuring cdx key is in SURT canonicalized.

* ``wb-manager autoindex `` / ``wayback -a`` -- Support for auto-updating the cdx indexes whenever any WARC/ARC files are modified or created.

* Switch default ``wayback``,  ``cdx-server``, ``live-rewrite-server`` cli apps to use ``waitress`` WSGI container instead of wsgi ref.

  New cli options, including ``-p`` (port), ``-t`` (num threads), and ``-d`` (working directory)

* url rewrite: fixes to JS url rewrite (some urls with unencoded chars were not being rewritten),
  fixes to WbUrl parsing of urls starting with digits (eg. 1234.example.com) not being parsed properly.

* framed replay: update frame_insert.html to be html5 compliant.

* wombat: fixed to WB_wombat_location.href assignment, properly redirects to dest page even if url is already rewritten

* static paths: static content included with pywb moved from ``static/default`` -> ``static/__pywb`` to free up default as possible collection name
  and avoid any naming conflicts. For example, wombat.js can be accessed via ``/static/__pywb/wombat.js``

* default to replay with framed mode enabled: ``framed_replay: true``


pywb 0.8.3 changelist
~~~~~~~~~~~~~~~~~~~~~

* cookie rewrite: all cookie rewriters remove ``secure`` flag to allow equivalent replay of sites with cookies via HTTP and HTTPS.

* html rewrite: fix ``<base>`` tag rewriting to add a trailing slash to the url if it is a hostname with no path, ex:

  ``<base href="http://example.com" />`` -> ``<base href="http://localhost:8080/rewrite/http://example.com/" />``

* framed replay: fix double slash that remainded when rewriting top frame url.


pywb 0.8.2 changelist
~~~~~~~~~~~~~~~~~~~~~

* rewrite: fix for redirect loop related to pages with 'www.' prefix. Since canonicalization removes the prefix, treat redirect to 'www.' as self-redirect (for now).

* memento: ensure rel=memento url matches timegate redirect exactly (urls may differ due to canonicalization, use actual instead of requested for both)


pywb 0.8.1 changelist
~~~~~~~~~~~~~~~~~~~~~

* wb.js top frame notification: use ``window.__orig_parent`` when referencing actual parent as ``window.parent`` now overriden.

* live proxy security: enable ssl verification for live proxy by default, for use with python 2.7.9 ssl improvements. Was disabled
  due to incomplete ssl support in previous versions of python. Can be disabled via ``verify_ssl: False`` per collection.

* cdx-indexer: add recursive option to index warcs in all subdirectories with ``cdx-indexer -r <dir_name>``


pywb 0.8.0 changelist
~~~~~~~~~~~~~~~~~~~~~

Improvements to framed replay, memento support, IDN urls, and additional customization support in preparation for further config changes.

* Feature: Full support for 'non-exact' or sticky timestamp browsing in framed and non-framed mode.

  - setting ``redir_to_exact: False`` (per collection), no redirects will be issued to the exact timestamp of the capture.
    The user-specified timestamp will be preserved and the number of redirects will be reduced.

  - if no timestamp is present (latest-replay request), there is a redirect to the current time UTC timestamp,
    available via ``pywb.utils.timeutils.timestamp_now()`` function.

  - via head-insert, the exact request timestamp is provided as ``wbinfo.request_ts`` and accessible to the banner insert or the top frame when in framed mode.

* Frame Mode Replay Improvements, including:

  - wombat: modify ``window.parent`` and ``window.frameElement`` to hide top-level non replay frame.

  - memento improvements: add same memento headers to top-level frame to match replay frame to ensure top-level frame
    passes memento header validation.

  - frame mode uses the request timestamp instead of the capture timestamp to update frame url.
    By default, request timestamp == capture timestamp, unless ``redir_to_exact: False`` (see above).

* Client-Side Rewrite Improvements:

  - improved ``document.write`` override to also work when in ``<head>`` and append both ``<head>`` and ``<body>``

  - detect multiple calls to rewrite attribute to avoid rewrite loops.

* Customization improvements:

  - ability to override global UrlRewriter with custom class by setting ``urlrewriter_class`` config setting.

  - ability to disable JS url and location rewrite via ``js_rewrite_location: none`` setting.

  - ability to set a custom content loader in place of default ARC/WARC loader in ``ReplayView._init_replay_view``

* Improved Memento compatibility, ensuring all responses have a ``rel=memento`` link.

* IDN support: Improved handling of non-ascii domains.

  - all urls are internally converted to a Punycode host, percent encoded path using IDNA encoding (http://tools.ietf.org/html/rfc3490.html).
  - when rendering, return convert all urls to fully percent-encoded by default (to allow browser to convert to unicode characters).
  - ``punycode_links`` rewrite option can be enabled to keep ascii-punycode hostnames instead of percent-encoding.


pywb 0.7.8 changelist
~~~~~~~~~~~~~~~~~~~~~

* live rewrite fix: When forwarding ``X-Forwarded-Proto`` header, set scheme to actual url scheme to avoid possible redirect loops (#57)


pywb 0.7.7 changelist
~~~~~~~~~~~~~~~~~~~~~

* client-side rewrite: improved rewriting of all style changes using mutation observers

* rules: fix YT rewrite rule, add rule for wikimedia

* cdx-indexer: minor cleanup, add support for custom writer for batched cdx (write_multi_cdx_index)


pywb 0.7.6 changelist
~~~~~~~~~~~~~~~~~~~~~

* new not found Jinja2 template: Add per-collection-overridable ``not_found.html`` template, specified via ``not_found_html`` option. For missing resources, the ``not_found_html`` template is now used instead of the generic ``error_html``

* client-side rewrite: improved wombat rewrite of postMessage events, unrewrite target on receive, improved Vine replay

* packaging: allow adding multiple packages for Jinja2 template resolving

pywb 0.7.5 changelist
~~~~~~~~~~~~~~~~~~~~~

* Cross platform fixes to support Windows -- all tests pass on Linux, OS X and Windows now. Improved cross-platform support includes:

  - read all files as binary to avoid line ending issues
  - properly convert between platform dependent file paths and urls
  - add .gitattributes to ensure line endings on *.warc*, *.arc*, *.cdx* files are unaltered
  - avoid platform dependent apis (eg. %s for strftime)

* Change any unhandled exceptions to result in a 500 error, instead of 400.

* Setup: switch to ``zip_safe=True`` to allow for embedding pywb egg in one-file app with `pyinstaller <https://github.com/pyinstaller/pyinstaller>`_

* More compresensive client side ``src`` attribute rewriting (via wombat.js), additional server-side HTML tag rewriting.


pywb 0.7.2 changelist
~~~~~~~~~~~~~~~~~~~~~

* Experiment with disabling DASH for YT

* New ``req_cookie_rewrite`` rewrite directive to rewrite outgoing ``Cookie`` header, can be used to fix a certain cookie for a url prefix.

  A list of regex match/replace rules, applied in succession, can be set for each url prefix. See ``rules.yaml`` for more info.


pywb 0.7.1 changelist
~~~~~~~~~~~~~~~~~~~~~

* (0.7.1 fixes some missing static files from 0.7.0 release)

* Video/Audio Replay, Live Proxy and Recording Support (with pywb-webrecorder)!

  See: `Video Replay and Recording <https://github.com/ikreymer/pywb/wiki/Video-Replay-and-Recording>`_ for more detailed info.

* Support for replaying HTTP/1.1 range requests for any archived resorce (optional range cache be disabled via `enable_ranges: false`)

* Support for on-the-fly video replacement of Flash with HTML5 using new video rewrite system ``vidrw.js``.

  (Designed for all Flash videos, with varying levels of special cases for YouTube, Vimeo, Soundcloud and Dailymotion)

* Use `youtube-dl <http://rg3.github.io/youtube-dl/>`_ to find actual video streams from page urls, record video info.

* New, improved wombat 2.1 -- improved rewriting of dynamic content, including:

  - setAttribute override
  - Date override sets date to replay timestamp
  - Image() object override
  - ability to disable dynamic attribute rewriting by setting ``_no_rewrite`` on an element.

* Type detection: resolve conflict between text/html that is served under js_ mod, resolve if html or js.


pywb 0.6.6 changelist
~~~~~~~~~~~~~~~~~~~~~

* JS client side improvements: check for double-inits, preserve anchor in wb.js top location redirect

* JS Rewriters: add mixins for link + location (default), link only, location only rewriting by setting ``js_rewrite_location`` to ``all``, ``urls``, ``location``, respectively.

  (New: location only rewriting does not change JS urls)

* Beginning of new rewrite options, settable per collections and stored in UrlRewriter. Available options:

  - ``rewrite_base`` - set to False to disable rewriting ``<base href="...">`` tag
  - ``rewrite_rel_canon`` - set to false to disable rewriting ``<link rel=canon href="...">``

* JS rewrite: Don't rewrite location if starting with '$'


pywb 0.6.5 changelist
~~~~~~~~~~~~~~~~~~~~~

* fix static handling when content type can not be guessed, default to 'application/octet-stream'

* rewrite fix: understand partially encoded urls such as http%3A// in WbUrl, decode correctly

* rewrite fix: rewrite \/\/example.com and \\/\\/example.com in JS same as \\example.com

* cookies: add exact cookie rewriter which sets cookie to exact url only, never collection or host root

* don't rewrite rel=canonical links for services which rely on these

* cdx-indexer: Detect non-gzip chunk encoded .warc.gz/arc.gz archive files and show a meaningful
  error message explaining how to fix issue (uncompress and possibly use warctools warc2warc to recompress)


pywb 0.6.4 changelist
~~~~~~~~~~~~~~~~~~~~~

* Ignore bad multiline headers in warc.

* Rewrite fix: Don't parse html entities in HTML rewriter.

* Ensure cdx iterator closed when reeading.

* Rewrite fix: remove pywb prefix from any query params.

* Rewrite fix: better JS rewriting, avoid // comments when matching protocol-relative urls.

* WARC metadata and resource records include in cdx from cdx-indexer by default


pywb 0.6.3 changelist
~~~~~~~~~~~~~~~~~~~~~

* Minor fixes for extensability and support https://webrecorder.io, easier to override any request (handle_request), handle_replay or handle_query via WBHandler


pywb 0.6.2 changelist
~~~~~~~~~~~~~~~~~~~~~

* Invert framed replay paradigm: Canonical page is always without a modifier (instead of with ``mp_``), if using frames, the page redirects to ``tf_``, and uses replaceState() to change url back to canonical form.

* Enable Memento support for framed replay, include Memento headers in top frame

* Easier to customize just the banner html, via ``banner_html`` setting in the config. Default banner uses ui/banner.html and inserts the script default_banner.js, which creates the banner.

  Other implementations may create banner via custom JS or directly insert HTML, as needed. Setting ``banner_html: False`` will disable the banner.

* Small improvements to streaming response, read in fixed chunks to allow better streaming from live.

* Improved cookie and csrf-token rewriting, including: ability to set ``cookie_scope: root`` per collection to have all replayed cookies have their Path set to application root.

  This is useful for replaying sites which share cookies amongst different pages and across archived time ranges.

* New, implified notation for fuzzy match rules on query params (See: `Fuzzy Match Rules <https://github.com/ikreymer/pywb/wiki/Fuzzy-Match-Rules>`_)


pywb 0.6.0 changelist
~~~~~~~~~~~~~~~~~~~~~

* HTTPS Proxy Support! (See: `Proxy Mode Usage <https://github.com/ikreymer/pywb/wiki/Pywb-Proxy-Mode-Usage>`_)

* Revamped HTTP/S system: proxy collection and capture time switching via cookie!

* removed *hostnames* setting in config.yaml. pywb no longer needs to know the host(s) it is running on,
  can infer the correct path from referrer on a fallback handling.

* remove PAC config, just using direct proxy (HTTP and HTTPS) for simplicity.


pywb 0.5.4 changelist
~~~~~~~~~~~~~~~~~~~~~

* bug fix: self-redirect check resolves relative Location: redirects

* rewrite rules: 'parse_comments' option to parse html comments as JS, regex rewrite update to match '&quot;http:\\\\/' double backslash

* bug fixes in framed replay for html content, update top frame for html content on load when possible


pywb 0.5.3 changelist
~~~~~~~~~~~~~~~~~~~~~
* better framed replay for non-html content -- include live rewrite timestamp via temp 'pywb.timestamp' cookie, updating banner of iframe load. All timestamp formatting moved to client-side for better customization.

* refactoring of replay/live handlers for better extensability.

* banner-only rewrite mode (via 'bn_' modifier) to support only banner insertion with no rewriting, server-side or client-side.


pywb 0.5.1 changelist
~~~~~~~~~~~~~~~~~~~~~
minor fixes:

* cdxindexer accepts unicode filenames, encodes via sys encoding

* SCRIPT_NAME now defaults to '' if not present


pywb 0.5.0 changelist
~~~~~~~~~~~~~~~~~~~~~

* Catch live rewrite errors and display more friendly pywb error message.

* LiveRewriteHandler and WBHandler refactoring: LiveRewriteHandler now supports a root search page html template.

* Proxy mode option: 'unaltered_replay' to proxy archival data with no modifications (no banner, no server or client side rewriting).

* Fix client side rewriting (wombat.js) for proxy mode: only rewrite https -> http in absolute urls.

* Fixes to memento timemap/timegate to work with framed replay mode.

* Support for a fallback handler which will be called from a replay handler instead of a 404 response.

  The handler, specified via the ``fallback`` option, can be the name of any other replay handler. Typically, it can be used with a live rewrite handler to fetch missing content from live instead of showing a 404.

* Live Rewrite can now be included as a 'collection type' in a pywb deployment by setting index path to ``$liveweb``.

* ``live-rewrite-server`` has optional ``--proxy host:port`` param to specify a loading live web data through an HTTP/S proxy, such as for use with a recording proxy.

* wombat: add document.cookie -> document.WB_wombat_cookie rewriting to check and rewrite Path= to archival url

* Better parent relative '../' path rewriting, resolved to correct absolute urls when rewritten. Additional testing for parent relative urls.

* New 'proxy_options' block, including 'use_default_coll' to allow defaulting to first collection w/o proxy auth.

* Improved support for proxy mode, allow different collections to be selected via proxy auth


pywb 0.4.7 changelist
~~~~~~~~~~~~~~~~~~~~~

* Tests: Additional testing of bad cdx lines, missing revisit records.

* Rewrite: Removal of lxml support for now, as it leads to problematic replay and not much performance improvements.

* Rewrite: Parsing of html as raw bytes instead of decode/encode, detection still needed for non-ascii compatible encoding.

* Indexing: Refactoring of cdx-indexer using a seperate 'archive record iterator' and pluggable cdx writer classes. Groundwork for creating custom indexers.

* Indexing: Support for 9 field cdx formats with -9 flag.

* Rewrite: Improved top -> WB_wombat_top rewriting.

* Rewrite: Better handling of framed replay url notification

pywb 0.4.5 changelist
~~~~~~~~~~~~~~~~~~~~~

* Support for framed or non-framed mode replay, toggleable via the ``framed_replay`` flag in the config.yaml

* Cookie rewriter: remove Max-Age to use ensure session-expiry instead of long-term cookie (experimental).

* Live Rewrite: proxy all headers, instead of a whitelist.

* Fixes to ``<base>`` tag handling, now correctly rewriting remainder of urls with the set base.

* ``cdx-indexer`` options for resolving POST requests, and indexing request records. (``-p`` and ``-a``)

* Improved `POST request replay <https://github.com/ikreymer/pywb/wiki/POST-request-replay>`_, allowing for improved replay of many captures relying on POST requests.

pywb 0.4.0 changelist
~~~~~~~~~~~~~~~~~~~~~

* Improved test coverage throughout the project.

* live-rewrite-server: A new web server for checking rewriting rules against live content. A white-list of request headers is sent to
  the destination server. See `rewrite_live.py <https://github.com/ikreymer/pywb/blob/master/pywb/rewrite/rewrite_live.py>`_ for more details.

* Cookie Rewriting in Archival Mode: HTTP Set-Cookie header rewritten to remove Expires, rewrite Path and Domain. If Domain is used, Path is set to / to ensure cookie is visible from all archival urls.

* Much improved handling of chunk encoded responses, better handling of zero-length chunks and fix bug where not enough gzip data was read for a full chunk to be decoded. Support for chunk-decoding w/o gzip decompression
  (for example, for binary data).

* Redis CDX: Initial support for reading entire CDX 'file' from a redis key via ZRANGEBYLEX, though needs more testing.

* Jinja templates: additional keyword args added to most templates for customization, export 'urlsplit' to use by templates.

* Remove SeekableLineReader, just using standard file-like object for binary search.

* Proper handling of js_ cs_ modifiers to select content-type.

* New, experimental support for top-level 'frame mode', used by live-rewrite-server, to display rewritten content in a frame. The mp_ modifier is used
  to indicate the main page when top-level page is a frame.

* cdx-indexer: Support for creation of non-SURT, url-ordered as well SURT-ordered CDX files.

* Further rewrite of wombat.js: support for window.open, postMessage overrides, additional rewriting at Node creation time, better hash change detection.
  Use ``Object.defineProperty`` whenever possible to better override assignment to various JS properties.
  See `wombat.js <https://github.com/ikreymer/pywb/blob/master/pywb/static/wombat.js>`_ for more info.

* Update wombat.js to support: scheme-relative urls rewriting, dom manipulation rewriting, disable web Worker api which could leak to live requests

* Fixed support for empty arc/warc records. Indexed with '-', replay with '204 No Content'

* Improve lxml rewriting, letting lxml handle parsing and decoding from bytestream directly (to address #36)


pywb 0.3.0 changelist
~~~~~~~~~~~~~~~~~~~~~

* Generate cdx indexs via command-line `cdx-indexer` script. Optionally sorting, and output to either a single combined file or a file per-directory.
  Refer to ``cdx-indexer -h`` for more info.

* Initial support for prefix url queries, eg: http://localhost:8080/pywb/\*/http://example.com\* to query all captures from http://example.com

* Support for optional LXML html-based parser for fastest possible parsing. If lxml is installed on the system and via ``pip install lxml``, lxml parser is enabled by default.
  (This can be turned off by setting ``use_lxml_parser: false`` in the config)

* Full support for `Memento Protocol RFC7089 <http://www.mementoweb.org/guide/rfc/>`_ Memento, TimeGate and TimeMaps. Memento: TimeMaps in ``application/link-format`` provided via the ``/timemap/*/`` query.. eg: http://localhost:8080/pywb/timemap/\*/http://example.com

* pywb now features new `domain-specific rules <https://github.com/ikreymer/pywb/blob/master/pywb/rules.yaml>`_ which are applied to resolve and render certain difficult and dynamic content, in order to make accurate web replay work.
  This ruleset will be under further iteration to address further challenges as the web evoles.
