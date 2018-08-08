---
layout: post
title:  "bootstrap下多层模态框的实现"
categories: Bootstrap
tags: Bootstrap
---

* content
{:toc}

最近有个点餐的后台页面要实现一个订单详情的弹框一个确认送货的弹框，两层的，基于bootstrap写的，众所周知bootstrap不支持两个及以上的modal，所以查了下，发现重写了关闭方法以及显示的方法之后还是可以实现的。




#### 主体

只要将下列方法加入页面后续执行，其他modal的调用方式和样式都不变，就能直接达到多层不冲突，黑色模态也不加深的效果。

```js
jQuery(function($){
        //解决模态框背景色越来越深的问题
        $(document).on('show.bs.modal', '.modal', function(event) {
            $(this).appendTo($('body'));
        }).on('shown.bs.modal', '.modal.in', function(event) {
            setModalsAndBackdropsOrder();
        }).on('hidden.bs.modal', '.modal', function(event) {
            setModalsAndBackdropsOrder();
        });

        function setModalsAndBackdropsOrder() {
            var modalZIndex = 1040;
            $('.modal.in').each(function(index) {
                var $modal = $(this);
                modalZIndex++;
                $modal.css('zIndex', modalZIndex);
                $modal.next('.modal-backdrop.in').addClass('hidden').css('zIndex', modalZIndex - 1);
            });
            $('.modal.in:visible:last').focus().next('.modal-backdrop.in').removeClass('hidden');
        }

        //覆盖Modal.prototype的hideModal方法
        $.fn.modal.Constructor.prototype.hideModal = function () {
            var that = this
            this.$element.hide()
            this.backdrop(function () {
                //判断当前页面所有的模态框都已经隐藏了之后body移除.modal-open，即body出现滚动条。
                $('.modal.fade.in').length === 0 && that.$body.removeClass('modal-open')
                that.resetAdjustments()
                that.resetScrollbar()
                that.$element.trigger('hidden.bs.modal')
            })
        }
    });
```

####  贴上所有代码

```html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>test2.html</title>
    <link href="/dddc1.0/css/bootstrap.min.css" rel="stylesheet">
    <style>
        #container{
            height:1000px;
        }
    </style>
</head>
<body>
<div id="container">
    <span>这是一个很长的div,使页面出现滚动条。</span>
    <h2>使用Bootstrap创建多模态框</h2>
    <div id="example1" class="modal fade">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <a class="close" data-dismiss="modal">×</a>
                    <h3>这是第一个模态框</h3>
                </div>
                <div class="modal-body">
                    <h4>第一个模态框中的文本</h4>
                    <p>你可以在这添加一些文本。</p>
                </div>
                <div class="modal-footer">
                    <a data-toggle="modal" href="#example2" class="btn btn-primary btn-large">弹出第二个模态框</a>
                    <a href="#" class="btn" data-dismiss="modal">关闭</a>
                </div>
            </div>
        </div>
    </div>
    <div id="example2" class="modal fade">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <a class="close" data-dismiss="modal">×</a>
                    <h3>这是第二个模态框</h3>
                </div>
                <div class="modal-body">
                    <h4>第二个模态框中的文本</h4>
                    <p style="height:900px">你可以在这添加一些文本。</p>
                </div>
                <div class="modal-footer">
                    <a data-toggle="modal" href="#example3" class="btn btn-primary btn-large">弹出第三个模态框</a>
                    <a href="#" class="btn" data-dismiss="modal">关闭</a>
                </div>
            </div>
        </div>
    </div>
    <div id="example3" class="modal fade">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <a class="close" data-dismiss="modal">×</a>
                    <h3>这是第三个模态框</h3>
                </div>
                <div class="modal-body">
                    <h4>第三个模态框中的文本</h4>
                    <p>你可以在这添加一些文本。</p>
                </div>
                <div class="modal-footer">
                    <!--<a data-toggle="modal" href="#example2" class="btn btn-primary btn-large">发动演示模态框</a>-->
                    <a href="#" class="btn" data-dismiss="modal">关闭</a>
                </div>
            </div>
        </div>
    </div>
    <p><a data-toggle="modal" href="#example1" class="btn btn-primary btn-large">发动演示模态框</a></p>
</div>
<script src="/dddc1.0/js/jquery-2.2.0.js"></script>
<script src="/dddc1.0/js/bootstrap.min.js"></script>
<script>
    jQuery(function($){
        //解决模态框背景色越来越深的问题
        $(document).on('show.bs.modal', '.modal', function(event) {
            $(this).appendTo($('body'));
        }).on('shown.bs.modal', '.modal.in', function(event) {
            setModalsAndBackdropsOrder();
        }).on('hidden.bs.modal', '.modal', function(event) {
            setModalsAndBackdropsOrder();
        });

        function setModalsAndBackdropsOrder() {
            var modalZIndex = 1040;
            $('.modal.in').each(function(index) {
                var $modal = $(this);
                modalZIndex++;
                $modal.css('zIndex', modalZIndex);
                $modal.next('.modal-backdrop.in').addClass('hidden').css('zIndex', modalZIndex - 1);
            });
            $('.modal.in:visible:last').focus().next('.modal-backdrop.in').removeClass('hidden');
        }

        //覆盖Modal.prototype的hideModal方法
        $.fn.modal.Constructor.prototype.hideModal = function () {
            var that = this
            this.$element.hide()
            this.backdrop(function () {
                //判断当前页面所有的模态框都已经隐藏了之后body移除.modal-open，即body出现滚动条。
                $('.modal.fade.in').length === 0 && that.$body.removeClass('modal-open')
                that.resetAdjustments()
                that.resetScrollbar()
                that.$element.trigger('hidden.bs.modal')
            })
        }
    });
</script>
</body>
</html>

```
