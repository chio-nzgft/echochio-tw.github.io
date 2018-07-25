---
layout: post
title: opencart 加顯示發票功能
date: 2018-07-25
tags: opencart
---

Opencart 測試加功能
====================================================
左選單加個發票功能 (多了 sale/invoice )

admin/controller/common/column_left.php

```

                        // Sales
                        $sale = array();

                        if ($this->user->hasPermission('access', 'sale/order')) {
                                $sale[] = array(
                                        'name'     => $this->language->get('text_order'),
                                        'href'     => $this->url->link('sale/order', 'user_token=' . $this->session->data['user_token']),
                                        'children' => array()
                                );
                        }

                        if ($this->user->hasPermission('access', 'sale/invoice')) {
                                $sale[] = array(
                                        'name'     => $this->language->get('text_invoice'),
                                        'href'     => $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token']),
                                        'children' => array()
                                );
                        }


                        if ($this->user->hasPermission('access', 'sale/return')) {
                                $sale[] = array(
                                        'name'     => $this->language->get('text_return'),
                                        'href'     => $this->url->link('sale/return', 'user_token=' . $this->session->data['user_token']),
                                        'children' => array()
                                );
                        }

```

語言也要有 text_invoice 這參數

admin/language/zh-hk/common/column_left.php

```
$_['text_invoice']                  = '訂單發票';
```

那  sale/invoice 的 controller 是 ... 

admin/controller/sale/invoice.php

(複製 admin/controller/sale/order.php 去修改)

```
<?php
class ControllerSaleInvoice extends Controller {
        private $error = array();

        public function index() {
                $this->load->language('sale/invoice');

                $this->document->setTitle($this->language->get('heading_title'));

                $this->load->model('sale/invoice');

                $this->getList();
        }


        public function edit() {
                $this->load->language('sale/invoice');

                $this->document->setTitle($this->language->get('heading_title'));

                $this->load->model('sale/invoice');

                $this->getForm();
        }

.....

```
裡面用到 language('sale/invoice') ..

admin/language/zh-hk/sale/invoice.php

(複製 admin/language/zh-hk/sale/order.php 去修改 ... )

```
$_['heading_title']                    = '訂單發票';
....

```


裡面用到 model('sale/invoice') ...

(複製 admin/model/sale/order.php 去修改 ... 取 invoice_no 為主)

裡面用到 model('sale/invoice') ...


修修改改好大概長這樣

<img src="/images/posts/opencart/2.png">
