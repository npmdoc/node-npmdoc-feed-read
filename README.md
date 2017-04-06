# api documentation for  [feed-read (v0.0.1)](https://github.com/sentientwaffle/feed-read)  [![npm package](https://img.shields.io/npm/v/npmdoc-feed-read.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-feed-read) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-feed-read.svg)](https://travis-ci.org/npmdoc/node-npmdoc-feed-read)
#### a ATOM and RSS feed parser

[![NPM](https://nodei.co/npm/feed-read.png?downloads=true)](https://www.npmjs.com/package/feed-read)

[![apidoc](https://npmdoc.github.io/node-npmdoc-feed-read/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-feed-read_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-feed-read/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-feed-read/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-feed-read/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "sentientwaffle",
        "url": "http://sentientwaffle.github.com/"
    },
    "bugs": {
        "url": "https://github.com/sentientwaffle/feed-read/issues"
    },
    "dependencies": {
        "request": "2.x.x",
        "sax": "0.3.x",
        "underscore": "1.x.x"
    },
    "description": "a ATOM and RSS feed parser",
    "devDependencies": {
        "connect": "1.x.x",
        "mocha": "0.x.x",
        "should": "0.x.x"
    },
    "directories": {},
    "dist": {
        "shasum": "2da3934d7f15bbdbe574dcac5403821e4161638b",
        "tarball": "https://registry.npmjs.org/feed-read/-/feed-read-0.0.1.tgz"
    },
    "engines": {
        "node": ">=0.4.1"
    },
    "homepage": "https://github.com/sentientwaffle/feed-read",
    "keywords": [
        "atom",
        "rss",
        "feed",
        "parser"
    ],
    "main": "./index",
    "maintainers": [
        {
            "name": "sentientwaffle",
            "email": "sentientwaffle@gmail.com"
        }
    ],
    "name": "feed-read",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/sentientwaffle/feed-read.git"
    },
    "scripts": {
        "test": "mocha"
    },
    "version": "0.0.1"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module feed-read](#apidoc.module.feed-read)
1.  [function <span class="apidocSignatureSpan">feed-read.</span>atom (xml, source, callback)](#apidoc.element.feed-read.atom)
1.  [function <span class="apidocSignatureSpan">feed-read.</span>get (feed_url, callback)](#apidoc.element.feed-read.get)
1.  [function <span class="apidocSignatureSpan">feed-read.</span>identify (xml)](#apidoc.element.feed-read.identify)
1.  [function <span class="apidocSignatureSpan">feed-read.</span>rss (xml, source, callback)](#apidoc.element.feed-read.rss)



# <a name="apidoc.module.feed-read"></a>[module feed-read](#apidoc.module.feed-read)

#### <a name="apidoc.element.feed-read.atom"></a>[function <span class="apidocSignatureSpan">feed-read.</span>atom (xml, source, callback)](#apidoc.element.feed-read.atom)
- description and source-code
```javascript
atom = function (xml, source, callback) {
  if (!callback) return FeedRead.atom(xml, "", source);

  var parser   = new FeedParser()
    , articles = []
    // Info about the feed itself, not an article.
    , meta     = {source: source}
    // The current article.
    , article
    // The author for when no author is specified for the post.
    , default_author;


  parser.onopentag = function(tag) {
    if (tag.name == "entry") article = tag;
  };

  parser.onclosetag = function(tagname, current_tag) {
    if (tagname == "entry") {
      articles.push(article);
      article = null;
    } else if (tagname == "author" && !article) {
      default_author = child_data(current_tag, "name");
    } else if (tagname == "link" && current_tag.attributes.rel != "self") {
      meta.link || (meta.link = current_tag.attributes.href);
    } else if (tagname == "title" && !current_tag.parent.parent) {
      meta.name = current_tag.children[0];
    }
  };

  parser.onend = function() {
    callback(null, _.map(articles,
      function(art) {
        var author = child_by_name(art, "author");
        if (author) author = child_data(author, "name");

        var obj = {
            title:     child_data(art, "title")
          , content:   child_data(art, "content")
          , published: child_data(art, "published")
                    || child_data(art, "updated")
          , author:    author || default_author
          , link:      child_by_name(art, "link").attributes.href
          , feed:      meta
          };
        if (obj.published) obj.published = new Date(obj.published);
        return obj;
      }
    ));
  };

  parser.write(xml);
}
```
- example usage
```shell
...
    });

## 'feed.rss(rss_string, callback)'
Parse a string of XML as RSS.

The callback receives '(err, articles)'.

## 'feed.atom(atom_string, callback)'
Parse a string of XML as ATOM.

The callback receives '(err, articles)'.

## 'feed.identify(xml_string)' // => "atom", "rss", or false
Identify what type of feed the XML represents.
...
```

#### <a name="apidoc.element.feed-read.get"></a>[function <span class="apidocSignatureSpan">feed-read.</span>get (feed_url, callback)](#apidoc.element.feed-read.get)
- description and source-code
```javascript
get = function (feed_url, callback) {
  request(feed_url, function(err, res, body) {
    if (err) return callback(err);
    var type = FeedRead.identify(body);
    if (type == "atom") {
      FeedRead.atom(body, feed_url, callback);
    } else if (type == "rss") {
      FeedRead.rss(body, feed_url, callback);
    } else {
      return callback(new Error( "Body is not RSS or ATOM"
                                , body.substr(0, 30), "..."));
    }
  });
}
```
- example usage
```shell
...
var FeedRead = module.exports = function(feed_url, callback) {
if (feed_url instanceof Array) {
  var feed_urls = feed_url
    , articles  = [];
  var next = function(i) {
    var feed_url = feed_urls[i];
    if (!feed_url) return callback(null, articles);
    FeedRead.get(feed_url, function(err, _articles) {
      if (err) return callback(err);
      articles = articles.concat(_articles);
      next(i + 1);
    });
  };
  next(0);
} else {
...
```

#### <a name="apidoc.element.feed-read.identify"></a>[function <span class="apidocSignatureSpan">feed-read.</span>identify (xml)](#apidoc.element.feed-read.identify)
- description and source-code
```javascript
identify = function (xml) {
  if (/<rss /i.test(xml)) {
    return "rss";
  } else if (/<feed /i.test(xml)) {
    return "atom";
  } else {
    return false;
  }
}
```
- example usage
```shell
...
The callback receives '(err, articles)'.

## 'feed.atom(atom_string, callback)'
Parse a string of XML as ATOM.

The callback receives '(err, articles)'.

## 'feed.identify(xml_string)' // => "atom", "rss", or false
Identify what type of feed the XML represents.

Returns 'false' when it is neither RSS or ATOM.


# License
See LICENSE.
...
```

#### <a name="apidoc.element.feed-read.rss"></a>[function <span class="apidocSignatureSpan">feed-read.</span>rss (xml, source, callback)](#apidoc.element.feed-read.rss)
- description and source-code
```javascript
rss = function (xml, source, callback) {
  if (!callback) return FeedRead.rss(xml, "", source);

  var parser   = new FeedParser()
    , articles = []
    // Info about the feed itself, not an article.
    , meta     = {source: source}
    // The current article.
    , article;


  parser.onopentag = function(tag) {
    if (tag.name == "item") article = tag;
  };

  parser.onclosetag = function(tagname, current_tag) {
    if (tagname == "item") {
      articles.push(article);
      article = null;
    } else if (tagname == "channel") {
      meta.link || (meta.link = child_data(current_tag, "link"));
      meta.name = child_data(current_tag, "title");
    }
  };

  parser.onend = function() {
    callback(null, _.map(articles,
      function(art) {
        var obj = {
            title:     child_data(art, "title")
          , content:   scrub_html(child_data(art, "content:encoded"))
                    || scrub_html(child_data(art, "description"))
          , published: child_data(art, "pubDate")
          , author:    child_data(art, "author")
                    || child_data(art, "dc:creator")
          , link:      child_data(art, "link")
          , feed:      meta
          };
        if (obj.published) obj.published = new Date(obj.published);
        return obj;
      }
    ));
  };

  parser.write(xml);
}
```
- example usage
```shell
...
      //   * "link"      - The original article link (String).
      //   * "content"   - The HTML content of the article (String).
      //   * "published" - The date that the article was published (Date).
      //   * "feed"      - {name, source, link}
      //
    });

## 'feed.rss(rss_string, callback)'
Parse a string of XML as RSS.

The callback receives '(err, articles)'.

## 'feed.atom(atom_string, callback)'
Parse a string of XML as ATOM.
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
