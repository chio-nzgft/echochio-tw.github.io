---
layout: post
title: image zoom + select slideshow
date: 2017-10-05
tags: image zoom select slideshow
---
要找 image zoom + select slideshow 的功能 感覺這個不錯 ...保留起來

[![https://github.com/echochio-tw/image-zoom-with-select-slideshow](https://github.com/echochio-tw/image-zoom-with-select-slideshow)](https://github.com/echochio-tw/image-zoom-with-select-slideshow)

----------
```
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <meta http-equiv="imagetoolbar" content="no"/>
    <title>jQuery ElevateZoom-Plus - An image zoom plugin</title>
    <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.7/jquery.min.js"></script>
    <script type="text/javascript"
            src="https://cdnjs.cloudflare.com/ajax/libs/jquery-easing/1.3/jquery.easing.min.js"></script>
    <script type="text/javascript" src="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.2.1/src/jquery.ez-plus.js"></script>
    <link rel="stylesheet" type="text/css" href="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.2.1/css/jquery.ez-plus.css" media="s
creen" />
    <script type="text/javascript" src="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.2.1/demo/js/web.js?m=20100203"></script>
 <script type="text/javascript"
            src="https://cdn.rawgit.com/igorlino/fancybox-plus/1.3.6/src/jquery.fancybox-plus.js"></script>
    <link rel="stylesheet" type="text/css"
          href="https://cdn.rawgit.com/igorlino/fancybox-plus/1.3.6/css/jquery.fancybox-plus.css" media="screen"/>

    <!--[if IE 6]>
    <script src="https://cdn.rawgit.com/igorlino/DD_belatedPNG/0.0.8a/DD_belatedPNG_0.0.8a.js"></script>

    <![endif]-->
    <script src="https://cdn.rawgit.com/sorccu/cufon/1.09i/js/cufon.js" type="text/javascript"></script>
    <script src="https://cdn.rawgit.com/igorlino/fancybox-plus/1.3.6/demo/js/Museo_300_300.font.js"

    <script type="text/javascript" src="https://cdn.rawgit.com/igorlino/snippet-helper/1.0.1/src/snippet-helper.js"></script>
    <link type="text/css" rel="stylesheet" href="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.2.1/demo/css/prism.css"/>
    <script type="text/javascript" src="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.2.1/demo/js/prism.js"></script>
</head>
<body>
<div id="page" class="home">

    <div id="col_wrap">
        <div id="col_left" class="zoom-wrapper">

            <div class="zoom-left">
                <img style="max-width:450px" id="img_01"
                     src="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image1.jpg"
                     data-zoom-image="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image1.jpg"
                    />

                <div id="gal1" style="max-width:450px;float:left;">
                    <a href="#" class="elevatezoom-gallery" data-update=""
                       data-image="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image1.jpg"
                       data-zoom-image="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image1.jpg">
                        <img id="img_01"
                             src="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image1.jpg"
                             width="100"/></a>

                    <a href="#" class="elevatezoom-gallery"
                       data-image="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image2.jpg"
                       data-zoom-image="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image2.jpg"
                        ><img id=""
                              src="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image2.jpg"
                              width="100"/></a>

                    <a href="tester" class="elevatezoom-gallery"
                       data-image="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image3.jpg"
                       data-zoom-image="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image3.jpg">
                        <img id=""
                             src="https://cdn.rawgit.com/igorlino/elevatezoom-plus/1.1.6/demo/images/large/image3.jpg"
                             width="100"/>
                    </a>

                </div>
            </div>
            <div style="clear:both;"></div>
            <script type="text/javascript">
                $(document).ready(function () {
                    $("#zoom_06").ezPlus({
                        zoomType: "inner",
                        debug: true,
                        cursor: "crosshair"
                    });
                });
            </script>

            <div class="scripts view_script_05">

        <script type="text/javascript">
            var snippets = [
                {code:"code-ezp-01", ext:"html,js"},
                {code:"code-ezp-02", ext:"html"},
                {code:"code-ezp-03", ext:"html,js"}
            ];
            $(document).ready(function () {
                snippetHelper.loadSnippets(snippets);
            });
        </script>
    </div>
</div>
</body>
</html>
```
----------
