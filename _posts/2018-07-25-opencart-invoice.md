---
layout: post
title: opencart 加顯示發票功能
date: 2018-07-25
tags: opencart
---

Opencart 加功能 - 發票功能
===================================================
台灣發票是 -->  2 個英文 + 8 個數字

那發票的 invoice_prefix 可以利用成兩個英文首碼+8 個數字

但是我懶沒用 .... 哈哈 ... 理論是要用的 ....XD

首先去 資料庫將發票由 int 改為 string ... 這就不寫了
--------------------------------------------------

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

order_info_invoice.twig 裡面讓使用者可輸入 新的發票號碼 

admin/view/template/sale/order_info_invoice.twig

```
                <td>{{ text_invoice }}</td>
                <td id="invoice" class="text-right"><input type="text"  id="get-invoice" name="invoice" value="{{ invoice_no }}" /></td>
                <td style="width: 1%;" class="text-center">
                    <button id="button-invoice" data-loading-text="{{ text_loading }}" data-toggle="tooltip" title="{{ button_save }}" class="btn btn-success btn-xs"><i class="fa fa-cog"></i></button>
              </tr>
              <tr>
```

之後 新的發票號碼 傳回 controller (sale/invoice/createinvoiceno)

我debug 看輸入的 發票號碼 有沒有抓到 ... 加了 alert ... 最後沒拿掉 ...還好吧...

admin/view/template/sale/order_info_invoice.twig
```
$(document).delegate('#button-invoice', 'click', function() {
        alert("發票號碼 : "+document.getElementById('get-invoice').value);
        $.ajax({
                url: 'index.php?route=sale/invoice/createinvoiceno&user_token={{ user_token }}&order_id={{ order_id }}&InvoiceNo='+document.getElementById('get-invoice').value,
                dataType: 'json',
                beforeSend: function() {
                        $('#button-invoice').button('loading');
                },
                complete: function() {
                        $('#button-invoice').button('reset');
                },
                success: function(json) {
                        $('.alert-dismissible').remove();

                        if (json['error']) {
                                $('#content > .container-fluid').prepend('<div class="alert alert-danger alert-dismissible"><i class="fa fa-exclamation-circle"></i> ' + json['error'] + '</div>');
                        }

                        if (json['invoice_no']) {
                                $('#invoice').html(json['invoice_no']);

                                $('#button-invoice').replaceWith('<button disabled="disabled" class="btn btn-success btn-xs"><i class="fa fa-cog"></i></button>');
                        }
                },
                error: function(xhr, ajaxOptions, thrownError) {
                        alert(thrownError + "\r\n" + xhr.statusText + "\r\n" + xhr.responseText);
                }
        });
});
```

那發票丟到 controller (sale/invoice/createinvoiceno)

就是 invoice.php 內的 function createInvoiceNo

然後丟到 model (我用 sale_order 的 createInvoiceNo) 

傳入 order_id , invoice_no 放入資料庫

回傳 model 回傳的 invoice_no 到 view (用 ajax 的 json 回傳)

admin/controller/sale/invoice.php

```
          public function createInvoiceNo() {
                  $json = array();

                  if (isset($this->request->get['order_id'])) {
                          $order_id = $this->request->get['order_id'];
                          // $invoice_no = $this->request->get['InvoiceNo'];
                  } else {
                          $order_id = 0;
                  }


                  if (isset($this->request->get['InvoiceNo'])) {
                          $invoice_no = $this->request->get['InvoiceNo'];
                  } else {
                          $invoice_no = 'NO !!';
                  }


                  if (!$this->user->hasPermission('modify', 'sale/invoice')) {
                          $json['error'] = $this->language->get('error_permission');
                  } else {
                          $this->load->model('sale/order');

                          $invoice_no = $this->model_sale_order->createInvoiceNo($order_id , $invoice_no);

                          if ($invoice_no) {
                                  $json['invoice_no'] = $invoice_no;
                          } else {
                                  $json['error'] = $this->language->get('error_action');
                          }
                  }

                  $this->response->addHeader('Content-Type: application/json');
                  $this->response->setOutput(json_encode($json));
          }
```

發票丟到 model (sale/order/createInvoiceNo)

就是 order.php 內的 function createInvoiceNo

傳入 order_id , invoice_no 放入資料庫 

傳回 放入的 invoice_no 給 controller

```
        public function createInvoiceNo($order_id, $invoice_no) {

                        $this->db->query("UPDATE `" . DB_PREFIX . "order` SET invoice_no = '" . (string)$invoice_no . "', invoice_prefix = '" . $this->db->escape($order_info['invoice_prefix']) . "' WHERE order_id = '" . (int)$order_id . "'");

                        return $order_info['invoice_prefix'] . $invoice_no;
        }
```



修修改改好大概長這樣

<img src="/images/posts/opencart/2.png">

<img src="/images/posts/opencart/3.png">

<img src="/images/posts/opencart/4.png">

<img src="/images/posts/opencart/5.png">

<img src="/images/posts/opencart/6.png">

debug 改了些原始碼(我也忘了改了啥 ... 反正可工作) :

admin/controller/sale/invoice.php

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


        protected function getList() {
                if (isset($this->request->get['filter_order_id'])) {
                        $filter_order_id = $this->request->get['filter_order_id'];
                } else {
                        $filter_order_id = '';
                }

                if (isset($this->request->get['filter_customer'])) {
                        $filter_customer = $this->request->get['filter_customer'];
                } else {
                        $filter_customer = '';
                }

                if (isset($this->request->get['filter_order_status'])) {
                        $filter_order_status = $this->request->get['filter_order_status'];
                } else {
                        $filter_order_status = '';
                }

                if (isset($this->request->get['filter_order_status_id'])) {
                        $filter_order_status_id = $this->request->get['filter_order_status_id'];
                } else {
                        $filter_order_status_id = '';
                }

                if (isset($this->request->get['filter_total'])) {
                        $filter_total = $this->request->get['filter_total'];
                } else {
                        $filter_total = '';
                }

                if (isset($this->request->get['filter_date_added'])) {
                        $filter_date_added = $this->request->get['filter_date_added'];
                } else {
                        $filter_date_added = '';
                }

                if (isset($this->request->get['filter_invoice_no'])) {
                        $filter_invoice_no = $this->request->get['filter_invoice_no'];
                } else {
                        $filter_invoice_no = '';
                }

                if (isset($this->request->get['sort'])) {
                        $sort = $this->request->get['sort'];
                } else {
                        $sort = 'o.order_id';
                }

                if (isset($this->request->get['order'])) {
                        $order = $this->request->get['order'];
                } else {
                        $order = 'DESC';
                }

                if (isset($this->request->get['page'])) {
                        $page = $this->request->get['page'];
                } else {
                        $page = 1;
                }

                $url = '';

                if (isset($this->request->get['filter_order_id'])) {
                        $url .= '&filter_order_id=' . $this->request->get['filter_order_id'];
                }

                if (isset($this->request->get['filter_customer'])) {
                        $url .= '&filter_customer=' . urlencode(html_entity_decode($this->request->get['filter_customer'], ENT_QUOTES, 'UTF-8'));
                }

                if (isset($this->request->get['filter_order_status'])) {
                        $url .= '&filter_order_status=' . $this->request->get['filter_order_status'];
                }

                if (isset($this->request->get['filter_order_status_id'])) {
                        $url .= '&filter_order_status_id=' . $this->request->get['filter_order_status_id'];
                }

                if (isset($this->request->get['filter_total'])) {
                        $url .= '&filter_total=' . $this->request->get['filter_total'];
                }

                if (isset($this->request->get['filter_date_added'])) {
                        $url .= '&filter_date_added=' . $this->request->get['filter_date_added'];
                }

                if (isset($this->request->get['filter_invoice_no'])) {
                        $url .= '&filter_invoice_no=' . $this->request->get['filter_invoice_no'];
                }

                if (isset($this->request->get['sort'])) {
                        $url .= '&sort=' . $this->request->get['sort'];
                }

                if (isset($this->request->get['order'])) {
                        $url .= '&order=' . $this->request->get['order'];
                }

                if (isset($this->request->get['page'])) {
                        $url .= '&page=' . $this->request->get['page'];
                }

                $data['breadcrumbs'] = array();

                $data['breadcrumbs'][] = array(
                        'text' => $this->language->get('text_home'),
                        'href' => $this->url->link('common/dashboard', 'user_token=' . $this->session->data['user_token'])
                );

                $data['breadcrumbs'][] = array(
                        'text' => $this->language->get('heading_title'),
                        'href' => $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . $url)
                );

                $data['invoice'] = $this->url->link('sale/order/invoice', 'user_token=' . $this->session->data['user_token']);
                $data['shipping'] = $this->url->link('sale/order/shipping', 'user_token=' . $this->session->data['user_token']);
                //$data['add'] = $this->url->link('sale/order/add', 'user_token=' . $this->session->data['user_token'] . $url);
                //$data['delete'] = str_replace('&amp;', '&', $this->url->link('sale/order/delete', 'user_token=' . $this->session->data['user_token'] . $url));

                $data['orders'] = array();

                $filter_data = array(
                        'filter_order_id'        => $filter_order_id,
                        'filter_customer'            => $filter_customer,
                        'filter_order_status'    => $filter_order_status,
                        'filter_order_status_id' => $filter_order_status_id,
                        'filter_total'           => $filter_total,
                        'filter_date_added'      => $filter_date_added,
                        'filter_invoice_no'      => $filter_invoice_no,
                        'sort'                   => $sort,
                        'order'                  => $order,
                        'start'                  => ($page - 1) * $this->config->get('config_limit_admin'),
                        'limit'                  => $this->config->get('config_limit_admin')
                );

                $order_total = $this->model_sale_invoice->getTotalOrders($filter_data);

                $results = $this->model_sale_invoice->getOrders($filter_data);

                foreach ($results as $result) {
                        $data['orders'][] = array(
                                'order_id'      => $result['order_id'],
                                'customer'      => $result['customer'],
                                'order_status'  => $result['order_status'] ? $result['order_status'] : $this->language->get('text_missing'),
                                'total'         => $this->currency->format($result['total'], $result['currency_code'], $result['currency_value']),
                                'date_added'    => date($this->language->get('datetime_format'), strtotime($result['date_added'])),
                                'invoice_no'    => $result['invoice_no'],
                                'shipping_code' => $result['shipping_code'],
                                'view'          => $this->url->link('sale/invoice/info', 'user_token=' . $this->session->data['user_token'] . '&order_id=' . $result['order_id'] . $url),
                                'edit'          => $this->url->link('sale/invoice/edit', 'user_token=' . $this->session->data['user_token'] . '&order_id=' . $result['order_id'] . $url)
                        );
                }

                $data['user_token'] = $this->session->data['user_token'];

                if (isset($this->error['warning'])) {
                        $data['error_warning'] = $this->error['warning'];
                } else {
                        $data['error_warning'] = '';
                }

                if (isset($this->session->data['success'])) {
                        $data['success'] = $this->session->data['success'];

                        unset($this->session->data['success']);
                } else {
                        $data['success'] = '';
                }

                if (isset($this->request->post['selected'])) {
                        $data['selected'] = (array)$this->request->post['selected'];
                } else {
                        $data['selected'] = array();
                }

                $url = '';

                if (isset($this->request->get['filter_order_id'])) {
                        $url .= '&filter_order_id=' . $this->request->get['filter_order_id'];
                }

                if (isset($this->request->get['filter_customer'])) {
                        $url .= '&filter_customer=' . urlencode(html_entity_decode($this->request->get['filter_customer'], ENT_QUOTES, 'UTF-8'));
                }

                if (isset($this->request->get['filter_order_status'])) {
                        $url .= '&filter_order_status=' . $this->request->get['filter_order_status'];
                }

                if (isset($this->request->get['filter_order_status_id'])) {
                        $url .= '&filter_order_status_id=' . $this->request->get['filter_order_status_id'];
                }

                if (isset($this->request->get['filter_total'])) {
                        $url .= '&filter_total=' . $this->request->get['filter_total'];
                }

                if (isset($this->request->get['filter_date_added'])) {
                        $url .= '&filter_date_added=' . $this->request->get['filter_date_added'];
                }

                if (isset($this->request->get['filter_invoice_no'])) {
                        $url .= '&filter_invoice_no=' . $this->request->get['filter_invoice_no'];
                }

                if ($order == 'ASC') {
                        $url .= '&order=DESC';
                } else {
                        $url .= '&order=ASC';
                }

                if (isset($this->request->get['page'])) {
                        $url .= '&page=' . $this->request->get['page'];
                }

                $data['sort_order'] = $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . '&sort=o.order_id' . $url);
                $data['sort_customer'] = $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . '&sort=customer' . $url);
                $data['sort_status'] = $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . '&sort=order_status' . $url);
                $data['sort_total'] = $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . '&sort=o.total' . $url);
                $data['sort_date_added'] = $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . '&sort=o.date_added' . $url);
                $data['sort_invoice_no'] = $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . '&sort=o.invoice_no' . $url);

                $url = '';

                if (isset($this->request->get['filter_order_id'])) {
                        $url .= '&filter_order_id=' . $this->request->get['filter_order_id'];
                }

                if (isset($this->request->get['filter_customer'])) {
                        $url .= '&filter_customer=' . urlencode(html_entity_decode($this->request->get['filter_customer'], ENT_QUOTES, 'UTF-8'));
                }

                if (isset($this->request->get['filter_order_status'])) {
                        $url .= '&filter_order_status=' . $this->request->get['filter_order_status'];
                }

                if (isset($this->request->get['filter_order_status_id'])) {
                        $url .= '&filter_order_status_id=' . $this->request->get['filter_order_status_id'];
                }

                if (isset($this->request->get['filter_total'])) {
                        $url .= '&filter_total=' . $this->request->get['filter_total'];
                }

                if (isset($this->request->get['filter_date_added'])) {
                        $url .= '&filter_date_added=' . $this->request->get['filter_date_added'];
                }

                if (isset($this->request->get['filter_invoice_no'])) {
                        $url .= '&filter_invoice_no=' . $this->request->get['filter_invoice_no'];
                }

                if (isset($this->request->get['sort'])) {
                        $url .= '&sort=' . $this->request->get['sort'];
                }

                if (isset($this->request->get['order'])) {
                        $url .= '&order=' . $this->request->get['order'];
                }

                $pagination = new Pagination();
                $pagination->total = $order_total;
                $pagination->page = $page;
                $pagination->limit = $this->config->get('config_limit_admin');
                $pagination->url = $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . $url . '&page={page}');

                $data['pagination'] = $pagination->render();

                $data['results'] = sprintf($this->language->get('text_pagination'), ($order_total) ? (($page - 1) * $this->config->get('config_limit_admin')) + 1 : 0, ((($page - 1) * $this->config->get('config_limit_admin')) > ($order_total - $this->config->get('config_limit_admin'))) ? $order_total : ((($page - 1) * $this->config->get('config_limit_admin')) + $this->config->get('config_limit_admin')), $order_total, ceil($order_total / $this->config->get('config_limit_admin')));

                $data['filter_order_id'] = $filter_order_id;
                $data['filter_customer'] = $filter_customer;
                $data['filter_order_status'] = $filter_order_status;
                $data['filter_order_status_id'] = $filter_order_status_id;
                $data['filter_total'] = $filter_total;
                $data['filter_date_added'] = $filter_date_added;
                $data['filter_invoice_no'] = $filter_invoice_no;

                $data['sort'] = $sort;
                $data['order'] = $order;

                $this->load->model('localisation/order_status');

                $data['order_statuses'] = $this->model_localisation_order_status->getOrderStatuses();

                // API login
                $data['catalog'] = HTTP_CATALOG;

                // API login
                $this->load->model('user/api');

                $api_info = $this->model_user_api->getApi($this->config->get('config_api_id'));

                if ($api_info && $this->user->hasPermission('modify', 'sale/invoice')) {
                        $session = new Session($this->config->get('session_engine'), $this->registry);

                        $session->start();

                        $this->model_user_api->deleteApiSessionBySessonId($session->getId());

                        $this->model_user_api->addApiSession($api_info['api_id'], $session->getId(), $this->request->server['REMOTE_ADDR']);

                        $session->data['api_id'] = $api_info['api_id'];

                        $data['api_token'] = $session->getId();
                } else {
                        $data['api_token'] = '';
                }

                $data['header'] = $this->load->controller('common/header');
                $data['column_left'] = $this->load->controller('common/column_left');
                $data['footer'] = $this->load->controller('common/footer');

                $this->response->setOutput($this->load->view('sale/order_list_invoice', $data));
        }

        public function getForm() {
                $data['text_form'] = !isset($this->request->get['order_id']) ? $this->language->get('text_add') : $this->language->get('text_edit');

                $url = '';

                if (isset($this->request->get['filter_order_id'])) {
                        $url .= '&filter_order_id=' . $this->request->get['filter_order_id'];
                }

                if (isset($this->request->get['filter_customer'])) {
                        $url .= '&filter_customer=' . urlencode(html_entity_decode($this->request->get['filter_customer'], ENT_QUOTES, 'UTF-8'));
                }

                if (isset($this->request->get['filter_order_status'])) {
                        $url .= '&filter_order_status=' . $this->request->get['filter_order_status'];
                }

                if (isset($this->request->get['filter_order_status_id'])) {
                        $url .= '&filter_order_status_id=' . $this->request->get['filter_order_status_id'];
                }

                if (isset($this->request->get['filter_total'])) {
                        $url .= '&filter_total=' . $this->request->get['filter_total'];
                }

                if (isset($this->request->get['filter_date_added'])) {
                        $url .= '&filter_date_added=' . $this->request->get['filter_date_added'];
                }

                if (isset($this->request->get['filter_invoice_no'])) {
                        $url .= '&filter_invoice_no=' . $this->request->get['filter_invoice_no'];
                }

                if (isset($this->request->get['sort'])) {
                        $url .= '&sort=' . $this->request->get['sort'];
                }

                if (isset($this->request->get['order'])) {
                        $url .= '&order=' . $this->request->get['order'];
                }

                if (isset($this->request->get['page'])) {
                        $url .= '&page=' . $this->request->get['page'];
                }

                $data['breadcrumbs'] = array();

                $data['breadcrumbs'][] = array(
                        'text' => $this->language->get('text_home'),
                        'href' => $this->url->link('common/dashboard', 'user_token=' . $this->session->data['user_token'])
                );

                $data['breadcrumbs'][] = array(
                        'text' => $this->language->get('heading_title'),
                        'href' => $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . $url)
                );

                $data['cancel'] = $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . $url);

                $data['user_token'] = $this->session->data['user_token'];

                if (isset($this->request->get['order_id'])) {
                        $order_info = $this->model_sale_invoice->getOrder($this->request->get['order_id']);
                }

                if (!empty($order_info)) {
                        $data['order_id'] = (int)$this->request->get['order_id'];
                        $data['store_id'] = $order_info['store_id'];
                        $data['store_url'] = HTTP_CATALOG;

                        $data['customer'] = $order_info['customer'];
                        $data['customer_id'] = $order_info['customer_id'];
                        $data['customer_group_id'] = $order_info['customer_group_id'];
                        $data['fullname'] = $order_info['fullname'];
                        $data['email'] = $order_info['email'];
                        $data['telephone'] = $order_info['telephone'];
                        $data['account_custom_field'] = $order_info['custom_field'];

                        $this->load->model('customer/customer');

                        $data['addresses'] = $this->model_customer_customer->getAddresses($order_info['customer_id']);

                        $data['payment_fullname'] = $order_info['payment_fullname'];
                        $data['payment_company'] = $order_info['payment_company'];
                        $data['payment_address_1'] = $order_info['payment_address_1'];
                        $data['payment_address_2'] = $order_info['payment_address_2'];
                        $data['payment_city'] = $order_info['payment_city'];
                        $data['payment_postcode'] = $order_info['payment_postcode'];
                        $data['payment_country_id'] = $order_info['payment_country_id'];
                        $data['payment_zone_id'] = $order_info['payment_zone_id'];
                        $data['payment_custom_field'] = $order_info['payment_custom_field'];
                        $data['payment_method'] = $order_info['payment_method'];
                        $data['payment_code'] = $order_info['payment_code'];

                        $data['shipping_fullname'] = $order_info['shipping_fullname'];
                        $data['shipping_company'] = $order_info['shipping_company'];
                        $data['shipping_address_1'] = $order_info['shipping_address_1'];
                        $data['shipping_address_2'] = $order_info['shipping_address_2'];
                        $data['shipping_city'] = $order_info['shipping_city'];
                        $data['shipping_postcode'] = $order_info['shipping_postcode'];
                        $data['shipping_country_id'] = $order_info['shipping_country_id'];
                        $data['shipping_zone_id'] = $order_info['shipping_zone_id'];
                        $data['shipping_custom_field'] = $order_info['shipping_custom_field'];
                        $data['shipping_method'] = $order_info['shipping_method'];
                        $data['shipping_code'] = $order_info['shipping_code'];

                        // Products
                        $data['order_products'] = array();

                        $products = $this->model_sale_invoice->getOrderProducts($this->request->get['order_id']);

                        foreach ($products as $product) {
                                $data['order_products'][] = array(
                                        'product_id' => $product['product_id'],
                                        'name'       => $product['name'],
                                        'model'      => $product['model'],
                                        'option'     => $this->model_sale_invoice->getOrderOptions($this->request->get['order_id'], $product['order_product_id']),
                                        'quantity'   => $product['quantity'],
                                        'price'      => $product['price'],
                                        'total'      => $product['total'],
                                        'reward'     => $product['reward']
                                );
                        }

                        // Vouchers
                        $data['order_vouchers'] = $this->model_sale_invoice->getOrderVouchers($this->request->get['order_id']);

                        $data['coupon'] = '';
                        $data['voucher'] = '';
                        $data['reward'] = '';

                        $data['order_totals'] = array();

                        $order_totals = $this->model_sale_invoice->getOrderTotals($this->request->get['order_id']);

                        foreach ($order_totals as $order_total) {
                                // If coupon, voucher or reward points
                                $start = strpos($order_total['title'], '(') + 1;
                                $end = strrpos($order_total['title'], ')');

                                if ($start && $end) {
                                        $data[$order_total['code']] = substr($order_total['title'], $start, $end - $start);
                                }
                        }

                        $data['order_status_id'] = $order_info['order_status_id'];
                        $data['comment'] = $order_info['comment'];
                        $data['affiliate_id'] = $order_info['affiliate_id'];
                        $data['affiliate'] = $order_info['affiliate_fullname'];
                        $data['currency_code'] = $order_info['currency_code'];
                } else {
                        $data['order_id'] = 0;
                        $data['store_id'] = 0;
                        $data['store_url'] = HTTP_CATALOG;

                        $data['customer'] = '';
                        $data['customer_id'] = '';
                        $data['customer_group_id'] = $this->config->get('config_customer_group_id');
                        $data['fullname'] = '';
                        $data['email'] = '';
                        $data['telephone'] = '';
                        $data['customer_custom_field'] = array();

                        $data['addresses'] = array();

                        $data['payment_fullname'] = '';
                        $data['payment_company'] = '';
                        $data['payment_address_1'] = '';
                        $data['payment_address_2'] = '';
                        $data['payment_city'] = '';
                        $data['payment_postcode'] = '';
                        $data['payment_country_id'] = '';
                        $data['payment_zone_id'] = '';
                        $data['payment_custom_field'] = array();
                        $data['payment_method'] = '';
                        $data['payment_code'] = '';

                        $data['shipping_fullname'] = '';
                        $data['shipping_company'] = '';
                        $data['shipping_address_1'] = '';
                        $data['shipping_address_2'] = '';
                        $data['shipping_city'] = '';
                        $data['shipping_postcode'] = '';
                        $data['shipping_country_id'] = '';
                        $data['shipping_zone_id'] = '';
                        $data['shipping_custom_field'] = array();
                        $data['shipping_method'] = '';
                        $data['shipping_code'] = '';

                        $data['order_products'] = array();
                        $data['order_vouchers'] = array();
                        $data['order_totals'] = array();

                        $data['order_status_id'] = $this->config->get('config_order_status_id');
                        $data['comment'] = '';
                        $data['affiliate_id'] = '';
                        $data['affiliate'] = '';
                        $data['currency_code'] = $this->config->get('config_currency');

                        $data['coupon'] = '';
                        $data['voucher'] = '';
                        $data['reward'] = '';
                }

                // Stores
                $this->load->model('setting/store');

                $data['stores'] = array();

                $data['stores'][] = array(
                        'store_id' => 0,
                        'name'     => $this->language->get('text_default')
                );

                $results = $this->model_setting_store->getStores();

                foreach ($results as $result) {
                        $data['stores'][] = array(
                                'store_id' => $result['store_id'],
                                'name'     => $result['name']
                        );
                }

                // Customer Groups
                $this->load->model('customer/customer_group');

                $data['customer_groups'] = $this->model_customer_customer_group->getCustomerGroups();

                // Custom Fields
                $this->load->model('customer/custom_field');

                $data['custom_fields'] = array();

                $filter_data = array(
                        'filter_status'  => 1,
                        'sort'  => 'cf.sort_order',
                        'order' => 'ASC'
                );

                $custom_fields = $this->model_customer_custom_field->getCustomFields($filter_data);

                foreach ($custom_fields as $custom_field) {
                        $data['custom_fields'][] = array(
                                'custom_field_id'    => $custom_field['custom_field_id'],
                                'custom_field_value' => $this->model_customer_custom_field->getCustomFieldValues($custom_field['custom_field_id']),
                                'name'               => $custom_field['name'],
                                'value'              => $custom_field['value'],
                                'type'               => $custom_field['type'],
                                'location'           => $custom_field['location'],
                                'sort_order'         => $custom_field['sort_order']
                        );
                }

                $this->load->model('localisation/order_status');

                $data['order_statuses'] = $this->model_localisation_order_status->getOrderStatuses();

                $this->load->model('localisation/country');

                $data['countries'] = $this->model_localisation_country->getCountries();

                $this->load->model('localisation/currency');

                $data['currencies'] = $this->model_localisation_currency->getCurrencies();

                $data['voucher_min'] = $this->config->get('config_voucher_min');

                $this->load->model('sale/voucher_theme');

                $data['voucher_themes'] = $this->model_sale_voucher_theme->getVoucherThemes();

                // API login
                $data['catalog'] = HTTP_CATALOG;

                // API login
                $this->load->model('user/api');

                $api_info = $this->model_user_api->getApi($this->config->get('config_api_id'));

                if ($api_info && $this->user->hasPermission('modify', 'sale/invoice')) {
                        $session = new Session($this->config->get('session_engine'), $this->registry);

                        $session->start();

                        $this->model_user_api->deleteApiSessionBySessonId($session->getId());

                        $this->model_user_api->addApiSession($api_info['api_id'], $session->getId(), $this->request->server['REMOTE_ADDR']);

                        $session->data['api_id'] = $api_info['api_id'];

                        $data['api_token'] = $session->getId();
                } else {
                        $data['api_token'] = '';
                }

                $data['header'] = $this->load->controller('common/header');
                $data['column_left'] = $this->load->controller('common/column_left');
                $data['footer'] = $this->load->controller('common/footer');

                $this->response->setOutput($this->load->view('sale/order_form_invoice', $data));
        }

        public function info() {
                $this->load->model('sale/invoice');

                if (isset($this->request->get['order_id'])) {
                        $order_id = (int)$this->request->get['order_id'];
                } else {
                        $order_id = 0;
                }

                $order_info = $this->model_sale_invoice->getOrder($order_id);

                if ($order_info) {
                        $this->load->language('sale/invoice');

                        $this->document->setTitle($this->language->get('heading_title'));

                        $data['text_ip_add'] = sprintf($this->language->get('text_ip_add'), $this->request->server['REMOTE_ADDR']);
                        $data['text_order'] = sprintf($this->language->get('text_order'), $order_id);

                        $url = '';

                        if (isset($this->request->get['filter_order_id'])) {
                                $url .= '&filter_order_id=' . $this->request->get['filter_order_id'];
                        }

                        if (isset($this->request->get['filter_customer'])) {
                                $url .= '&filter_customer=' . urlencode(html_entity_decode($this->request->get['filter_customer'], ENT_QUOTES, 'UTF-8'));
                        }

                        if (isset($this->request->get['filter_order_status'])) {
                                $url .= '&filter_order_status=' . $this->request->get['filter_order_status'];
                        }

                        if (isset($this->request->get['filter_order_status_id'])) {
                                $url .= '&filter_order_status_id=' . $this->request->get['filter_order_status_id'];
                        }

                        if (isset($this->request->get['filter_total'])) {
                                $url .= '&filter_total=' . $this->request->get['filter_total'];
                        }

                        if (isset($this->request->get['filter_date_added'])) {
                                $url .= '&filter_date_added=' . $this->request->get['filter_date_added'];
                        }

                        if (isset($this->request->get['filter_invoice_no'])) {
                                $url .= '&filter_invoice_no=' . $this->request->get['filter_invoice_no'];
                        }

                        if (isset($this->request->get['sort'])) {
                                $url .= '&sort=' . $this->request->get['sort'];
                        }

                        if (isset($this->request->get['order'])) {
                                $url .= '&order=' . $this->request->get['order'];
                        }

                        if (isset($this->request->get['page'])) {
                                $url .= '&page=' . $this->request->get['page'];
                        }

                        $data['breadcrumbs'] = array();

                        $data['breadcrumbs'][] = array(
                                'text' => $this->language->get('text_home'),
                                'href' => $this->url->link('common/dashboard', 'user_token=' . $this->session->data['user_token'])
                        );

                        $data['breadcrumbs'][] = array(
                                'text' => $this->language->get('heading_title'),
                                'href' => $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . $url)
                        );

                        $data['shipping'] = $this->url->link('sale/order/shipping', 'user_token=' . $this->session->data['user_token'] . '&order_id=' . (int)$this->request->get['order_id']);
                        $data['invoice'] = $this->url->link('sale/order/invoice', 'user_token=' . $this->session->data['user_token'] . '&order_id=' . (int)$this->request->get['order_id']);
                        $data['edit'] = $this->url->link('sale/invoice/edit', 'user_token=' . $this->session->data['user_token'] . '&order_id=' . (int)$this->request->get['order_id']);
                        $data['cancel'] = $this->url->link('sale/invoice', 'user_token=' . $this->session->data['user_token'] . $url);

                        $data['user_token'] = $this->session->data['user_token'];

                        $data['order_id'] = (int)$this->request->get['order_id'];

                        $data['store_id'] = $order_info['store_id'];
                        $data['store_name'] = $order_info['store_name'];

                        if ($order_info['store_id'] == 0) {
                                $data['store_url'] = HTTP_CATALOG;
                        } else {
                                $data['store_url'] = $order_info['store_url'];
                        }

                        if ($order_info['invoice_no']) {
                                $data['invoice_no'] = $order_info['invoice_prefix'] . $order_info['invoice_no'];
                        } else {
                                $data['invoice_no'] = '';
                        }

                        $data['date_added'] = date($this->language->get('datetime_format'), strtotime($order_info['date_added']));

                        $data['fullname'] = $order_info['fullname'];

                        if ($order_info['customer_id']) {
                                $data['customer'] = $this->url->link('customer/customer/edit', 'user_token=' . $this->session->data['user_token'] . '&customer_id=' . $order_info['customer_id']);
                        } else {
                                $data['customer'] = '';
                        }

                        $this->load->model('customer/customer_group');

                        $customer_group_info = $this->model_customer_customer_group->getCustomerGroup($order_info['customer_group_id']);

                        if ($customer_group_info) {
                                $data['customer_group'] = $customer_group_info['name'];
                        } else {
                                $data['customer_group'] = '';
                        }

                        $data['email'] = $order_info['email'];
                        $data['telephone'] = $order_info['telephone'];

                        $data['shipping_method'] = $order_info['shipping_method'];
                        $data['payment_method'] = $order_info['payment_method'];

                        if (!empty($order_info['shipping_method'])) {
                                // Payment Address
                                $data['payment_address'] = address_format($order_info, $order_info['payment_address_format'], 'payment');

                                // Shipping Address
                                $data['shipping_address'] = address_format($order_info, $order_info['shipping_address_format'], 'shipping');
                        } else {
                                // Payment Address
                                $data['payment_address'] = '';

                                // Shipping Address
                                $data['shipping_address'] = '';
                        }

                        // Uploaded files
                        $this->load->model('tool/upload');

                        $data['products'] = array();

                        $products = $this->model_sale_invoice->getOrderProducts($this->request->get['order_id']);

                        foreach ($products as $product) {
                                $option_data = array();

                                $options = $this->model_sale_invoice->getOrderOptions($this->request->get['order_id'], $product['order_product_id']);

                                foreach ($options as $option) {
                                        if ($option['type'] != 'file') {
                                                $option_data[] = array(
                                                        'name'  => $option['name'],
                                                        'value' => $option['value'],
                                                        'type'  => $option['type']
                                                );
                                        } else {
                                                $upload_info = $this->model_tool_upload->getUploadByCode($option['value']);

                                                if ($upload_info) {
                                                        $option_data[] = array(
                                                                'name'  => $option['name'],
                                                                'value' => $upload_info['name'],
                                                                'type'  => $option['type'],
                                                                'href'  => $this->url->link('tool/upload/download', 'user_token=' . $this->session->data['user_token'] . '&code=' . $upload_info['code'])
                                                        );
                                                }
                                        }
                                }

                                $data['products'][] = array(
                                        'order_product_id' => $product['order_product_id'],
                                        'product_id'       => $product['product_id'],
                                        'name'                     => $product['name'],
                                        'model'                    => $product['model'],
                                        'option'                   => $option_data,
                                        'quantity'                 => $product['quantity'],
                                        'price'                    => $this->currency->format($product['price'] + ($this->config->get('config_tax') ? $product['tax'] : 0), $order_info['currency_code'], $order_info['currency_value']),
                                        'total'                    => $this->currency->format($product['total'] + ($this->config->get('config_tax') ? ($product['tax'] * $product['quantity']) : 0), $order_info['currency_code'], $order_info['currency_value']),
                                        'href'                     => $this->url->link('catalog/product/edit', 'user_token=' . $this->session->data['user_token'] . '&product_id=' . $product['product_id'])
                                );
                        }

                        $data['vouchers'] = array();

                        $vouchers = $this->model_sale_invoice->getOrderVouchers($this->request->get['order_id']);

                        foreach ($vouchers as $voucher) {
                                $data['vouchers'][] = array(
                                        'description' => $voucher['description'],
                                        'amount'      => $this->currency->format($voucher['amount'], $order_info['currency_code'], $order_info['currency_value']),
                                        'href'        => $this->url->link('sale/voucher/edit', 'user_token=' . $this->session->data['user_token'] . '&voucher_id=' . $voucher['voucher_id'])
                                );
                        }

                        $data['totals'] = array();

                        $totals = $this->model_sale_invoice->getOrderTotals($this->request->get['order_id']);

                        foreach ($totals as $total) {
                                $data['totals'][] = array(
                                        'title' => $total['title'],
                                        'text'  => $this->currency->format($total['value'], $order_info['currency_code'], $order_info['currency_value'])
                                );
                        }

                        $data['comment'] = nl2br($order_info['comment']);

                        $this->load->model('customer/customer');

                        $data['reward'] = $order_info['reward'];

                        $data['reward_total'] = $this->model_customer_customer->getTotalCustomerRewardsByOrderId($this->request->get['order_id']);

                        $data['affiliate_fullname'] = $order_info['affiliate_fullname'];

                        if ($order_info['affiliate_id']) {
                                $data['affiliate'] = $this->url->link('customer/customer/edit', 'user_token=' . $this->session->data['user_token'] . '&customer_id=' . $order_info['affiliate_id']);
                        } else {
                                $data['affiliate'] = '';
                        }

                        $data['commission'] = $this->currency->format($order_info['commission'], $order_info['currency_code'], $order_info['currency_value']);

                        $this->load->model('customer/customer');

                        $data['commission_total'] = $this->model_customer_customer->getTotalTransactionsByOrderId($this->request->get['order_id']);

                        $this->load->model('localisation/order_status');

                        $order_status_info = $this->model_localisation_order_status->getOrderStatus($order_info['order_status_id']);

                        if ($order_status_info) {
                                $data['order_status'] = $order_status_info['name'];
                        } else {
                                $data['order_status'] = '';
                        }

                        $data['order_statuses'] = $this->model_localisation_order_status->getOrderStatuses();

                        $data['order_status_id'] = $order_info['order_status_id'];

                        $data['account_custom_field'] = $order_info['custom_field'];

                        // Custom Fields
                        $this->load->model('customer/custom_field');

                        $data['account_custom_fields'] = array();

                        $filter_data = array(
                                'sort'  => 'cf.sort_order',
                                'order' => 'ASC'
                        );

                        $custom_fields = $this->model_customer_custom_field->getCustomFields($filter_data);

                        foreach ($custom_fields as $custom_field) {
                                if ($custom_field['location'] == 'account' && isset($order_info['custom_field'][$custom_field['custom_field_id']])) {
                                        if ($custom_field['type'] == 'select' || $custom_field['type'] == 'radio') {
                                                $custom_field_value_info = $this->model_customer_custom_field->getCustomFieldValue($order_info['custom_field'][$custom_field['custom_field_id']]);

                                                if ($custom_field_value_info) {
                                                        $data['account_custom_fields'][] = array(
                                                                'name'  => $custom_field['name'],
                                                                'value' => $custom_field_value_info['name']
                                                        );
                                                }
                                        }

                                        if ($custom_field['type'] == 'checkbox' && is_array($order_info['custom_field'][$custom_field['custom_field_id']])) {
                                                foreach ($order_info['custom_field'][$custom_field['custom_field_id']] as $custom_field_value_id) {
                                                        $custom_field_value_info = $this->model_customer_custom_field->getCustomFieldValue($custom_field_value_id);

                                                        if ($custom_field_value_info) {
                                                                $data['account_custom_fields'][] = array(
                                                                        'name'  => $custom_field['name'],
                                                                        'value' => $custom_field_value_info['name']
                                                                );
                                                        }
                                                }
                                        }

                                        if ($custom_field['type'] == 'text' || $custom_field['type'] == 'textarea' || $custom_field['type'] == 'file' || $custom_field['type'] == 'date' || $custom_field['type'] == 'datetime' || $custom_field['type'] == 'time') {
                                                $data['account_custom_fields'][] = array(
                                                        'name'  => $custom_field['name'],
                                                        'value' => $order_info['custom_field'][$custom_field['custom_field_id']]
                                                );
                                        }

                                        if ($custom_field['type'] == 'file') {
                                                $upload_info = $this->model_tool_upload->getUploadByCode($order_info['custom_field'][$custom_field['custom_field_id']]);

                                                if ($upload_info) {
                                                        $data['account_custom_fields'][] = array(
                                                                'name'  => $custom_field['name'],
                                                                'value' => $upload_info['name']
                                                        );
                                                }
                                        }
                                }
                        }

                        // Custom fields
                        $data['payment_custom_fields'] = array();

                        foreach ($custom_fields as $custom_field) {
                                if ($custom_field['location'] == 'address' && isset($order_info['payment_custom_field'][$custom_field['custom_field_id']])) {
                                        if ($custom_field['type'] == 'select' || $custom_field['type'] == 'radio') {
                                                $custom_field_value_info = $this->model_customer_custom_field->getCustomFieldValue($order_info['payment_custom_field'][$custom_field['custom_field_id']]);

                                                if ($custom_field_value_info) {
                                                        $data['payment_custom_fields'][] = array(
                                                                'name'       => $custom_field['name'],
                                                                'value'      => $custom_field_value_info['name'],
                                                                'sort_order' => $custom_field['sort_order']
                                                        );
                                                }
                                        }

                                        if ($custom_field['type'] == 'checkbox' && is_array($order_info['payment_custom_field'][$custom_field['custom_field_id']])) {
                                                foreach ($order_info['payment_custom_field'][$custom_field['custom_field_id']] as $custom_field_value_id) {
                                                        $custom_field_value_info = $this->model_customer_custom_field->getCustomFieldValue($custom_field_value_id);

                                                        if ($custom_field_value_info) {
                                                                $data['payment_custom_fields'][] = array(
                                                                        'name'       => $custom_field['name'],
                                                                        'value'      => $custom_field_value_info['name'],
                                                                        'sort_order' => $custom_field['sort_order']
                                                                );
                                                        }
                                                }
                                        }

                                        if ($custom_field['type'] == 'text' || $custom_field['type'] == 'textarea' || $custom_field['type'] == 'file' || $custom_field['type'] == 'date' || $custom_field['type'] == 'datetime' || $custom_field['type'] == 'time') {
                                                $data['payment_custom_fields'][] = array(
                                                        'name'       => $custom_field['name'],
                                                        'value'      => $order_info['payment_custom_field'][$custom_field['custom_field_id']],
                                                        'sort_order' => $custom_field['sort_order']
                                                );
                                        }

                                        if ($custom_field['type'] == 'file') {
                                                $upload_info = $this->model_tool_upload->getUploadByCode($order_info['payment_custom_field'][$custom_field['custom_field_id']]);

                                                if ($upload_info) {
                                                        $data['payment_custom_fields'][] = array(
                                                                'name'       => $custom_field['name'],
                                                                'value'      => $upload_info['name'],
                                                                'sort_order' => $custom_field['sort_order']
                                                        );
                                                }
                                        }
                                }
                        }

                        // Shipping
                        $data['shipping_custom_fields'] = array();

                        foreach ($custom_fields as $custom_field) {
                                if ($custom_field['location'] == 'address' && isset($order_info['shipping_custom_field'][$custom_field['custom_field_id']])) {
                                        if ($custom_field['type'] == 'select' || $custom_field['type'] == 'radio') {
                                                $custom_field_value_info = $this->model_customer_custom_field->getCustomFieldValue($order_info['shipping_custom_field'][$custom_field['custom_field_id']]);

                                                if ($custom_field_value_info) {
                                                        $data['shipping_custom_fields'][] = array(
                                                                'name'       => $custom_field['name'],
                                                                'value'      => $custom_field_value_info['name'],
                                                                'sort_order' => $custom_field['sort_order']
                                                        );
                                                }
                                        }

                                        if ($custom_field['type'] == 'checkbox' && is_array($order_info['shipping_custom_field'][$custom_field['custom_field_id']])) {
                                                foreach ($order_info['shipping_custom_field'][$custom_field['custom_field_id']] as $custom_field_value_id) {
                                                        $custom_field_value_info = $this->model_customer_custom_field->getCustomFieldValue($custom_field_value_id);

                                                        if ($custom_field_value_info) {
                                                                $data['shipping_custom_fields'][] = array(
                                                                        'name'       => $custom_field['name'],
                                                                        'value'      => $custom_field_value_info['name'],
                                                                        'sort_order' => $custom_field['sort_order']
                                                                );
                                                        }
                                                }
                                        }

                                        if ($custom_field['type'] == 'text' || $custom_field['type'] == 'textarea' || $custom_field['type'] == 'file' || $custom_field['type'] == 'date' || $custom_field['type'] == 'datetime' || $custom_field['type'] == 'time') {
                                                $data['shipping_custom_fields'][] = array(
                                                        'name'       => $custom_field['name'],
                                                        'value'      => $order_info['shipping_custom_field'][$custom_field['custom_field_id']],
                                                        'sort_order' => $custom_field['sort_order']
                                                );
                                        }

                                        if ($custom_field['type'] == 'file') {
                                                $upload_info = $this->model_tool_upload->getUploadByCode($order_info['shipping_custom_field'][$custom_field['custom_field_id']]);

                                                if ($upload_info) {
                                                        $data['shipping_custom_fields'][] = array(
                                                                'name'       => $custom_field['name'],
                                                                'value'      => $upload_info['name'],
                                                                'sort_order' => $custom_field['sort_order']
                                                        );
                                                }
                                        }
                                }
                        }

                        $data['ip'] = $order_info['ip'];
                        $data['forwarded_ip'] = $order_info['forwarded_ip'];
                        $data['user_agent'] = $order_info['user_agent'];
                        $data['accept_language'] = $order_info['accept_language'];

                        // Additional Tabs
                        $data['tabs'] = array();

                        if ($this->user->hasPermission('access', 'extension/payment/' . $order_info['payment_code'])) {
                                if (is_file(DIR_CATALOG . 'controller/extension/payment/' . $order_info['payment_code'] . '.php')) {
                                        $content = $this->load->controller('extension/payment/' . $order_info['payment_code'] . '/order');
                                } else {
                                        $content = '';
                                }

                                if ($content) {
                                        $this->load->language('extension/payment/' . $order_info['payment_code']);

                                        $data['tabs'][] = array(
                                                'code'    => $order_info['payment_code'],
                                                'title'   => $this->language->get('heading_title'),
                                                'content' => $content
                                        );
                                }
                        }

                        $this->load->model('setting/extension');

                        $extensions = $this->model_setting_extension->getInstalled('fraud');

                        foreach ($extensions as $extension) {
                                if ($this->config->get('fraud_' . $extension . '_status')) {
                                        $this->load->language('extension/fraud/' . $extension, 'extension');

                                        $content = $this->load->controller('extension/fraud/' . $extension . '/order');

                                        if ($content) {
                                                $data['tabs'][] = array(
                                                        'code'    => $extension,
                                                        'title'   => $this->language->get('extension')->get('heading_title'),
                                                        'content' => $content
                                                );
                                        }
                                }
                        }

                        // The URL we send API requests to
                        $data['catalog'] = HTTP_CATALOG;

                        // API login
                        $this->load->model('user/api');

                        $api_info = $this->model_user_api->getApi($this->config->get('config_api_id'));

                        if ($api_info && $this->user->hasPermission('modify', 'sale/invoice')) {
                                $session = new Session($this->config->get('session_engine'), $this->registry);

                                $session->start();

                                $this->model_user_api->deleteApiSessionBySessonId($session->getId());

                                $this->model_user_api->addApiSession($api_info['api_id'], $session->getId(), $this->request->server['REMOTE_ADDR']);

                                $session->data['api_id'] = $api_info['api_id'];

                                $data['api_token'] = $session->getId();
                        } else {
                                $data['api_token'] = '';
                        }

                        $data['header'] = $this->load->controller('common/header');
                        $data['column_left'] = $this->load->controller('common/column_left');
                        $data['footer'] = $this->load->controller('common/footer');

                        $this->response->setOutput($this->load->view('sale/order_info_invoice', $data));
                } else {
                        return new Action('error/not_found');
                }
        }

        protected function validate() {
                if (!$this->user->hasPermission('modify', 'sale/invoice')) {
                        $this->error['warning'] = $this->language->get('error_permission');
                }

                return !$this->error;
        }

        public function createInvoiceNo() {
                $json = array();

                if (isset($this->request->get['order_id'])) {
                        $order_id = $this->request->get['order_id'];
                        // $invoice_no = $this->request->get['InvoiceNo'];
                } else {
                        $order_id = 0;
                }


                if (isset($this->request->get['InvoiceNo'])) {
                        $invoice_no = $this->request->get['InvoiceNo'];
                } else {
                        $invoice_no = 'NO !!';
                }


                if (!$this->user->hasPermission('modify', 'sale/invoice')) {
                        $json['error'] = $this->language->get('error_permission');
                } else {
                        $this->load->model('sale/order');

                        $invoice_no = $this->model_sale_order->createInvoiceNo($order_id , $invoice_no);

                        if ($invoice_no) {
                                $json['invoice_no'] = $invoice_no;
                        } else {
                                $json['error'] = $this->language->get('error_action');
                        }
                }

                $this->response->addHeader('Content-Type: application/json');
                $this->response->setOutput(json_encode($json));
        }

        public function addReward() {
                $this->load->language('sale/invoice');

                $json = array();

                if (isset($this->request->get['order_id'])) {
                        $order_id = $this->request->get['order_id'];
                } else {
                        $order_id = 0;
                }

                if (!$this->user->hasPermission('modify', 'sale/invoice')) {
                        $json['error'] = $this->language->get('error_permission');
                } else {
                        $this->load->model('sale/invoice');

                        $order_info = $this->model_sale_order->getOrder($order_id);

                        if ($order_info && $order_info['customer_id'] && ($order_info['reward'] > 0)) {
                                $this->load->model('customer/customer');

                                $reward_total = $this->model_customer_customer->getTotalCustomerRewardsByOrderId($order_id);

                                if (!$reward_total) {
                                        $this->model_customer_customer->addReward($order_info['customer_id'], $this->language->get('text_order_id') . ' #' . $order_id, $order_info['reward'], $order_id);
                                }
                        }

                        $json['success'] = $this->language->get('text_reward_added');
                }

                $this->response->addHeader('Content-Type: application/json');
                $this->response->setOutput(json_encode($json));
        }

        public function removeReward() {
                $this->load->language('sale/invoice');

                $json = array();

                if (isset($this->request->get['order_id'])) {
                        $order_id = $this->request->get['order_id'];
                } else {
                        $order_id = 0;
                }

                if (!$this->user->hasPermission('modify', 'sale/invoice')) {
                        $json['error'] = $this->language->get('error_permission');
                } else {
                        $this->load->model('sale/invoice');

                        $order_info = $this->model_sale_order->getOrder($order_id);

                        if ($order_info) {
                                $this->load->model('customer/customer');

                                $this->model_customer_customer->deleteReward($order_id);
                        }

                        $json['success'] = $this->language->get('text_reward_removed');
                }

                $this->response->addHeader('Content-Type: application/json');
                $this->response->setOutput(json_encode($json));
        }

        public function addCommission() {
                $this->load->language('sale/invoice');

                $json = array();

                if (isset($this->request->get['order_id'])) {
                        $order_id = $this->request->get['order_id'];
                } else {
                        $order_id = 0;
                }

                if (!$this->user->hasPermission('modify', 'sale/invoice')) {
                        $json['error'] = $this->language->get('error_permission');
                } else {
                        $this->load->model('sale/invoice');

                        $order_info = $this->model_sale_order->getOrder($order_id);

                        if ($order_info) {
                                $this->load->model('customer/customer');

                                $affiliate_total = $this->model_customer_customer->getTotalTransactionsByOrderId($order_id);

                                if (!$affiliate_total) {
                                        $this->model_customer_customer->addTransaction($order_info['affiliate_id'], $this->language->get('text_order_id') . ' #' . $order_id, $order_info['commission'], $order_id);
                                }
                        }

                        $json['success'] = $this->language->get('text_commission_added');
                }

                $this->response->addHeader('Content-Type: application/json');
                $this->response->setOutput(json_encode($json));
        }

        public function removeCommission() {
                $this->load->language('sale/invoice');

                $json = array();

                if (isset($this->request->get['order_id'])) {
                        $order_id = $this->request->get['order_id'];
                } else {
                        $order_id = 0;
                }

                if (!$this->user->hasPermission('modify', 'sale/invoice')) {
                        $json['error'] = $this->language->get('error_permission');
                } else {
                        $this->load->model('sale/invoice');

                        $order_info = $this->model_sale_order->getOrder($order_id);

                        if ($order_info) {
                                $this->load->model('customer/customer');

                                $this->model_customer_customer->deleteTransactionByOrderId($order_id);
                        }

                        $json['success'] = $this->language->get('text_commission_removed');
                }

                $this->response->addHeader('Content-Type: application/json');
                $this->response->setOutput(json_encode($json));
        }

        public function history() {
                $this->load->language('sale/invoice');

                if (isset($this->request->get['order_id'])) {
                        $order_id = $this->request->get['order_id'];
                } else {
                        $order_id = 0;
                }

                if (isset($this->request->get['page'])) {
                        $page = $this->request->get['page'];
                } else {
                        $page = 1;
                }

                $data['histories'] = array();

                $this->load->model('sale/invoice');

                $results = $this->model_sale_order->getOrderHistories($order_id, ($page - 1) * 10, 10);

                foreach ($results as $result) {
                        $data['histories'][] = array(
                                'notify'     => $result['notify'] ? $this->language->get('text_yes') : $this->language->get('text_no'),
                                'status'     => $result['status'],
                                'comment'    => nl2br($result['comment']),
                                'date_added' => date($this->language->get('datetime_format'), strtotime($result['date_added']))
                        );
                }

                $history_total = $this->model_sale_order->getTotalOrderHistories($order_id);

                $pagination = new Pagination();
                $pagination->total = $history_total;
                $pagination->page = $page;
                $pagination->limit = 10;
                $pagination->url = $this->url->link('sale/order/history', 'user_token=' . $this->session->data['user_token'] . '&order_id=' . $order_id . '&page={page}');

                $data['pagination'] = $pagination->render();

                $data['results'] = sprintf($this->language->get('text_pagination'), ($history_total) ? (($page - 1) * 10) + 1 : 0, ((($page - 1) * 10) > ($history_total - 10)) ? $history_total : ((($page - 1) * 10) + 10), $history_total, ceil($history_total / 10));

                $this->response->setOutput($this->load->view('sale/order_history', $data));
        }

        public function invoice() {
                $this->load->language('sale/order');

                $data['title'] = $this->language->get('text_invoice');

                $data['base'] = HTTP_SERVER;
                $data['direction'] = $this->language->get('direction');
                $data['lang'] = $this->language->get('code');

                $this->load->model('sale/order');

                $this->load->model('setting/setting');

                $data['orders'] = array();

                $orders = array();

                if (isset($this->request->post['selected'])) {
                        $orders = $this->request->post['selected'];
                } elseif (isset($this->request->get['order_id'])) {
                        $orders[] = $this->request->get['order_id'];
                }

                foreach ($orders as $order_id) {
                        $order_info = $this->model_sale_order->getOrder($order_id);

                        if ($order_info) {
                                $store_info = $this->model_setting_setting->getSetting('config', $order_info['store_id']);

                                if ($store_info) {
                                        $store_address = $store_info['config_address'];
                                        $store_email = $store_info['config_email'];
                                        $store_telephone = $store_info['config_telephone'];
                                        $store_fax = $store_info['config_fax'];
                                } else {
                                        $store_address = $this->config->get('config_address');
                                        $store_email = $this->config->get('config_email');
                                        $store_telephone = $this->config->get('config_telephone');
                                        $store_fax = $this->config->get('config_fax');
                                }

                                if ($order_info['invoice_no']) {
                                        $invoice_no = $order_info['invoice_prefix'] . $order_info['invoice_no'];
                                } else {
                                        $invoice_no = '';
                                }

                                if (!empty($order_info['payment_method'])) {
                                        $payment_address = address_format($order_info, $order_info['payment_address_format'], 'payment');
                                        $shipping_address = address_format($order_info, $order_info['shipping_address_format'], 'shipping');
                                } else {
                                        $payment_address = '';
                                        $shipping_address = '';
                                }

                                $this->load->model('tool/upload');

                                $product_data = array();

                                $products = $this->model_sale_order->getOrderProducts($order_id);

                                foreach ($products as $product) {
                                        $option_data = array();

                                        $options = $this->model_sale_order->getOrderOptions($order_id, $product['order_product_id']);

                                        foreach ($options as $option) {
                                                if ($option['type'] != 'file') {
                                                        $value = $option['value'];
                                                } else {
                                                        $upload_info = $this->model_tool_upload->getUploadByCode($option['value']);

                                                        if ($upload_info) {
                                                                $value = $upload_info['name'];
                                                        } else {
                                                                $value = '';
                                                        }
                                                }

                                                $option_data[] = array(
                                                        'name'  => $option['name'],
                                                        'value' => $value
                                                );
                                        }

                                        $product_data[] = array(
                                                'name'     => $product['name'],
                                                'model'    => $product['model'],
                                                'option'   => $option_data,
                                                'quantity' => $product['quantity'],
                                                'price'    => $this->currency->format($product['price'] + ($this->config->get('config_tax') ? $product['tax'] : 0), $order_info['currency_code'], $order_info['currency_value']),
                                                'total'    => $this->currency->format($product['total'] + ($this->config->get('config_tax') ? ($product['tax'] * $product['quantity']) : 0), $order_info['currency_code'], $order_info['currency_value'])
                                        );
                                }

                                $voucher_data = array();

                                $vouchers = $this->model_sale_order->getOrderVouchers($order_id);

                                foreach ($vouchers as $voucher) {
                                        $voucher_data[] = array(
                                                'description' => $voucher['description'],
                                                'amount'      => $this->currency->format($voucher['amount'], $order_info['currency_code'], $order_info['currency_value'])
                                        );
                                }

                                $total_data = array();

                                $totals = $this->model_sale_order->getOrderTotals($order_id);

                                foreach ($totals as $total) {
                                        $total_data[] = array(
                                                'title' => $total['title'],
                                                'text'  => $this->currency->format($total['value'], $order_info['currency_code'], $order_info['currency_value'])
                                        );
                                }

                                $data['orders'][] = array(
                                        'order_id'             => $order_id,
                                        'invoice_no'       => $invoice_no,
                                        'date_added'       => date($this->language->get('datetime_format'), strtotime($order_info['date_added'])),
                                        'store_name'       => $order_info['store_name'],
                                        'store_url'        => rtrim($order_info['store_url'], '/'),
                                        'store_address'    => nl2br($store_address),
                                        'store_email'      => $store_email,
                                        'store_telephone'  => $store_telephone,
                                        'store_fax'        => $store_fax,
                                        'email'            => $order_info['email'],
                                        'telephone'        => $order_info['telephone'],
                                        'shipping_address' => $shipping_address,
                                        'shipping_method'  => $order_info['shipping_method'],
                                        'payment_address'  => $payment_address,
                                        'payment_method'   => $order_info['payment_method'],
                                        'product'          => $product_data,
                                        'voucher'          => $voucher_data,
                                        'total'            => $total_data,
                                        'comment'          => nl2br($order_info['comment'])
                                );
                        }
                }

                $this->response->setOutput($this->load->view('sale/order_invoice', $data));
        }

        public function shipping() {
                $this->load->language('sale/order');

                $data['title'] = $this->language->get('text_shipping');

                $data['base'] = HTTP_SERVER;
                $data['direction'] = $this->language->get('direction');
                $data['lang'] = $this->language->get('code');

                $this->load->model('sale/order');

                $this->load->model('catalog/product');
                $this->load->model('catalog/product_option');

                $this->load->model('setting/setting');

                $data['orders'] = array();

                $orders = array();

                if (isset($this->request->post['selected'])) {
                        $orders = $this->request->post['selected'];
                } elseif (isset($this->request->get['order_id'])) {
                        $orders[] = $this->request->get['order_id'];
                }

                foreach ($orders as $order_id) {
                        $order_info = $this->model_sale_order->getOrder($order_id);

                        // Make sure there is a shipping method
                        if ($order_info && $order_info['shipping_code']) {
                                $store_info = $this->model_setting_setting->getSetting('config', $order_info['store_id']);

                                if ($store_info) {
                                        $store_address = $store_info['config_address'];
                                        $store_email = $store_info['config_email'];
                                        $store_telephone = $store_info['config_telephone'];
                                } else {
                                        $store_address = $this->config->get('config_address');
                                        $store_email = $this->config->get('config_email');
                                        $store_telephone = $this->config->get('config_telephone');
                                }

                                if ($order_info['invoice_no']) {
                                        $invoice_no = $order_info['invoice_prefix'] . $order_info['invoice_no'];
                                } else {
                                        $invoice_no = '';
                                }

                                $shipping_address = address_format($order_info, $order_info['shipping_address_format'], 'shipping');

                                $this->load->model('tool/upload');

                                $product_data = array();

                                $products = $this->model_sale_order->getOrderProducts($order_id);

                                foreach ($products as $product) {
                                        $option_weight = 0;

                                        $product_info = $this->model_catalog_product->getProduct($product['product_id']);

                                        if ($product_info) {
                                                $option_data = array();

                                                $options = $this->model_sale_order->getOrderOptions($order_id, $product['order_product_id']);

                                                foreach ($options as $option) {
                                                        if ($option['type'] != 'file') {
                                                                $value = $option['value'];
                                                        } else {
                                                                $upload_info = $this->model_tool_upload->getUploadByCode($option['value']);

                                                                if ($upload_info) {
                                                                        $value = $upload_info['name'];
                                                                } else {
                                                                        $value = '';
                                                                }
                                                        }

                                                        $option_data[] = array(
                                                                'name'  => $option['name'],
                                                                'value' => $value
                                                        );

                                                        $product_option_value_info = $this->model_catalog_product_option->getProductOptionValue($product['product_id'], $option['product_option_value_id']);

                                                        if ($product_option_value_info) {
                                                                if ($product_option_value_info['weight_prefix'] == '+') {
                                                                        $option_weight += $product_option_value_info['weight'];
                                                                } elseif ($product_option_value_info['weight_prefix'] == '-') {
                                                                        $option_weight -= $product_option_value_info['weight'];
                                                                }
                                                        }
                                                }

                                                $product_data[] = array(
                                                        'name'     => $product_info['name'],
                                                        'model'    => $product_info['model'],
                                                        'option'   => $option_data,
                                                        'quantity' => $product['quantity'],
                                                        'location' => $product_info['location'],
                                                        'sku'      => $product_info['sku'],
                                                        'upc'      => $product_info['upc'],
                                                        'ean'      => $product_info['ean'],
                                                        'jan'      => $product_info['jan'],
                                                        'isbn'     => $product_info['isbn'],
                                                        'mpn'      => $product_info['mpn'],
                                                        'weight'   => $this->weight->format(($product_info['weight'] + (float)$option_weight) * $product['quantity'], $product_info['weight_class_id'], $this->language->get('decimal_point'), $this->language->get('thousand_point'))
                                                );
                                        }
                                }

                                $data['orders'][] = array(
                                        'order_id'             => $order_id,
                                        'invoice_no'       => $invoice_no,
                                        'date_added'       => date($this->language->get('datetime_format'), strtotime($order_info['date_added'])),
                                        'store_name'       => $order_info['store_name'],
                                        'store_url'        => rtrim($order_info['store_url'], '/'),
                                        'store_address'    => nl2br($store_address),
                                        'store_email'      => $store_email,
                                        'store_telephone'  => $store_telephone,
                                        'email'            => $order_info['email'],
                                        'telephone'        => $order_info['telephone'],
                                        'shipping_address' => $shipping_address,
                                        'shipping_method'  => $order_info['shipping_method'],
                                        'product'          => $product_data,
                                        'comment'          => nl2br($order_info['comment'])
                                );
                        }
                }

                $this->response->setOutput($this->load->view('sale/order_shipping', $data));
        }
}
```
admin/view/template/sale/order_list_invoice.twig

```
{{ header }}{{ column_left }}
<div id="content">
  <div class="page-header">
    <div class="container-fluid">
      <div class="pull-right">
        <button type="button" data-toggle="tooltip" title="{{ button_filter }}" onclick="$('#filter-order').toggleClass('hidden-sm hidden-xs');" class="btn btn-default hidden-md hidden-lg"><i class="fa fa-filter"></i></button>
        <button type="submit" id="button-shipping" form="form-order" formaction="{{ shipping }}" formtarget="_blank" data-toggle="tooltip" title="{{ button_shipping_print }}" class="btn btn-info"><i class="fa fa-truck"></i></button>
        <button type="submit" id="button-invoice" form="form-order" formaction="{{ invoice }}" formtarget="_blank" data-toggle="tooltip" title="{{ button_invoice_print }}" class="btn btn-info"><i class="fa fa-print"></i></button>
        <data-toggle="tooltip" class="btn btn-primary"><i class="fa fa-plus"></i></a></div>
      <h1>{{ heading_title }}</h1>
      <ul class="breadcrumb">
        {% for breadcrumb in breadcrumbs %}
          <li><a href="{{ breadcrumb.href }}">{{ breadcrumb.text }}</a></li>
        {% endfor %}
      </ul>
    </div>
  </div>
  <div class="container-fluid">{% if error_warning %}
      <div class="alert alert-danger alert-dismissible"><i class="fa fa-exclamation-circle"></i> {{ error_warning }}
        <button type="button" class="close" data-dismiss="alert">&times;</button>
      </div>
    {% endif %}
    {% if success %}
      <div class="alert alert-success alert-dismissible"><i class="fa fa-check-circle"></i> {{ success }}
        <button type="button" class="close" data-dismiss="alert">&times;</button>
      </div>
    {% endif %}
    <div class="row">
      <div id="filter-order" class="col-md-3 col-md-push-9 col-sm-12 hidden-sm hidden-xs">
        <div class="panel panel-default">
          <div class="panel-heading">
            <h3 class="panel-title"><i class="fa fa-filter"></i> {{ text_filter }}</h3>
          </div>
          <div class="panel-body">
            <div class="form-group">
              <label class="control-label" for="input-order-id">{{ entry_order_id }}</label> <input type="text" name="filter_order_id" value="{{ filter_order_id }}" placeholder="{{ entry_order_id }}" id="input-order-id" class="form-control"/>
            </div>
            <div class="form-group">
              <label class="control-label" for="input-customer">{{ entry_customer }}</label> <input type="text" name="filter_customer" value="{{ filter_customer }}" placeholder="{{ entry_customer }}" id="input-customer" class="form-control"/>
            </div>
            <div class="form-group">
              <label class="control-label" for="input-order-status">{{ entry_order_status }}</label> <select name="filter_order_status_id" id="input-order-status" class="form-control">
                <option value=""></option>

                {% if filter_order_status_id == '0' %}

                  <option value="0" selected="selected">{{ text_missing }}</option>

                {% else %}

                  <option value="0">{{ text_missing }}</option>

                {% endif %}
                {% for order_status in order_statuses %}
                  {% if order_status.order_status_id == filter_order_status_id %}

                    <option value="{{ order_status.order_status_id }}" selected="selected">{{ order_status.name }}</option>

                  {% else %}

                    <option value="{{ order_status.order_status_id }}">{{ order_status.name }}</option>

                  {% endif %}
                {% endfor %}

              </select>
            </div>
            <div class="form-group">
              <label class="control-label" for="input-total">{{ entry_total }}</label>
              <input type="text" name="filter_total" value="{{ filter_total }}" placeholder="{{ entry_total_format }}" id="input-total" class="form-control"/>
            </div>
            <div class="form-group">
              <label class="control-label" for="input-date-added">{{ entry_date_added_start }}</label>
              <div class="input-group date">
                <input type="text" name="filter_date_added" value="{{ filter_date_added }}" placeholder="{{ entry_date_added_start }}" data-date-format="YYYY-MM-DD" id="input-date-added" class="form-control"/>
                <div class="input-group-btn">
                  <button type="button" class="btn btn-default"><i class="fa fa-calendar"></i></button>
                </div>
              </div>
            </div>
            <div class="form-group">
              <label class="control-label" for="input-date-modified">{{ entry_date_added_end }}</label>
              <div class="input-group date">
                <input type="text" name="filter_invoice_no" value="{{ filter_invoice_no }}" placeholder="{{ entry_date_added_end }}" data-date-format="YYYY-MM-DD" id="input-date-modified" class="form-control"/>
                <div class="input-group-btn">
                  <button type="button" class="btn btn-default"><i class="fa fa-calendar"></i></button>
                </div>
              </div>
            </div>
            <div class="form-group text-right">
              <button type="button" id="button-filter" class="btn btn-default"><i class="fa fa-filter"></i> {{ button_filter }}</button>
            </div>
          </div>
        </div>
      </div>
      <div class="col-md-9 col-md-pull-3 col-sm-12">
        <div class="panel panel-default">
          <div class="panel-heading">
            <h3 class="panel-title"><i class="fa fa-list"></i> {{ text_list }}</h3>
          </div>
          <div class="panel-body">
            <form method="post" action="" enctype="multipart/form-data" id="form-order">
              <div class="table-responsive">
                <table class="table table-bordered table-hover">
                  <thead>
                  <tr>
                    <td style="width: 1px;" class="text-center"><input type="checkbox" onclick="$('input[name*=\'selected\']').trigger('click');"/></td>
                    <td class="text-right">{% if sort == 'o.order_id' %} <a href="{{ sort_order }}" class="{{ order|lower }}">{{ column_order_id }}</a> {% else %} <a href="{{ sort_order }}">{{ column_order_id }}</a> {% endif %}</td>
                    <td class="text-left">{% if sort == 'customer' %} <a href="{{ sort_customer }}" class="{{ order|lower }}">{{ column_customer }}</a> {% else %} <a href="{{ sort_customer }}">{{ column_customer }}</a> {% endif %}</td>
                    <td class="text-left">{% if sort == 'order_status' %} <a href="{{ sort_status }}" class="{{ order|lower }}">{{ column_status }}</a> {% else %} <a href="{{ sort_status }}">{{ column_status }}</a> {% endif %}</td>
                    <td class="text-right">{% if sort == 'o.total' %} <a href="{{ sort_total }}" class="{{ order|lower }}">{{ column_total }}</a> {% else %} <a href="{{ sort_total }}">{{ column_total }}</a> {% endif %}</td>
                    <td class="text-left">{% if sort == 'o.date_added' %} <a href="{{ sort_date_added }}" class="{{ order|lower }}">{{ column_date_added }}</a> {% else %} <a href="{{ sort_date_added }}">{{ column_date_added }}</a> {% endif %}</td>
                    <td class="text-left">{% if sort == 'o.invoice_no' %} <a href="{{ sort_invoice_no }}" class="{{ order|lower }}">{{ column_invoice_no }}</a> {% else %} <a href="{{ sort_invoice_no }}">{{ column_invoice_no }}</a> {% endif %}</td>
                    <td class="text-right">{{ column_action }}</td>
                  </tr>
                  </thead>
                  <tbody>

                  {% if orders %}
                    {% for order in orders %}
                      <tr>
                        <td class="text-center">{% if order.order_id in selected %}
                            <input type="checkbox" name="selected[]" value="{{ order.order_id }}" checked="checked"/>
                          {% else %}
                            <input type="checkbox" name="selected[]" value="{{ order.order_id }}"/>
                          {% endif %}
                          <input type="hidden" name="shipping_code[]" value="{{ order.shipping_code }}"/></td>
                        <td class="text-right">{{ order.order_id }}</td>
                        <td class="text-left">{{ order.customer }}</td>
                        <td class="text-left">{{ order.order_status }}</td>
                        <td class="text-right">{{ order.total }}</td>
                        <td class="text-left">{{ order.date_added }}</td>
                        <td class="text-left">{{ order.invoice_no }}</td>
                        <td class="text-right">
                          <div style="min-width: 120px;">
                            <div class="btn-group"><a href="{{ order.view }}" data-toggle="tooltip" title="{{ button_view }}" class="btn btn-primary"><i class="fa fa-eye"></i></a>
                            </div>
                          </div>
                        </td>
                      </tr>
                    {% endfor %}
                  {% else %}
                    <tr>
                      <td class="text-center" colspan="8">{{ text_no_results }}</td>
                    </tr>
                  {% endif %}
                  </tbody>

                </table>
              </div>
            </form>
            <div class="row">
              <div class="col-sm-6 text-left">{{ pagination }}</div>
              <div class="col-sm-6 text-right">{{ results }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
<script type="text/javascript"><!--
$('#button-filter').on('click', function() {
        url = '';

        var filter_order_id = $('input[name=\'filter_order_id\']').val();

        if (filter_order_id) {
                url += '&filter_order_id=' + encodeURIComponent(filter_order_id);
        }

        var filter_customer = $('input[name=\'filter_customer\']').val();

        if (filter_customer) {
                url += '&filter_customer=' + encodeURIComponent(filter_customer);
        }

        var filter_order_status_id = $('select[name=\'filter_order_status_id\']').val();

        if (filter_order_status_id !== '') {
                url += '&filter_order_status_id=' + encodeURIComponent(filter_order_status_id);
        }

        var filter_total = $('input[name=\'filter_total\']').val();

        if (filter_total) {
                url += '&filter_total=' + encodeURIComponent(filter_total);
        }

        var filter_date_added = $('input[name=\'filter_date_added\']').val();

        if (filter_date_added) {
                url += '&filter_date_added=' + encodeURIComponent(filter_date_added);
        }

        var filter_invoice_no = $('input[name=\'filter_invoice_no\']').val();

        if (filter_invoice_no) {
                url += '&filter_invoice_no=' + encodeURIComponent(filter_invoice_no);
        }

        location = 'index.php?route=sale/invoice&user_token={{ user_token }}' + url;
});
//--></script>
<script type="text/javascript"><!--
$('input[name=\'filter_customer\']').autocomplete({
        'source': function(request, response) {
                $.ajax({
                        url: 'index.php?route=customer/customer/autocomplete&user_token={{ user_token }}&filter_name=' + encodeURIComponent(request),
                        dataType: 'json',
                        success: function(json) {
                                response($.map(json, function(item) {
                                        return {
                                                label: item['name'],
                                                value: item['customer_id']
                                        }
                                }));
                        }
                });
        },
        'select': function(item) {
                $('input[name=\'filter_customer\']').val(item['label']);
        }
});
//--></script>
<script type="text/javascript"><!--
$('input[name^=\'selected\']').on('change', function() {
        $('#button-shipping, #button-invoice').prop('disabled', true);

        var selected = $('input[name^=\'selected\']:checked');

        if (selected.length) {
                $('#button-invoice').prop('disabled', false);
        }

        for (i = 0; i < selected.length; i++) {
                if ($(selected[i]).parent().find('input[name^=\'shipping_code\']').val()) {
                        $('#button-shipping').prop('disabled', false);

                        break;
                }
        }
});

$('#button-shipping, #button-invoice').prop('disabled', true);

$('input[name^=\'selected\']:first').trigger('change');

// IE and Edge fix!
$('#button-shipping, #button-invoice').on('click', function(e) {
        $('#form-order').attr('action', this.getAttribute('formAction'));
});

$('#form-order li:last-child a').on('click', function(e) {
        e.preventDefault();

        var element = this;

        if (confirm('{{ text_confirm }}')) {
                $.ajax({
                        url: '{{ catalog }}index.php?route=api/order/delete&api_token={{ api_token }}&store_id={{ store_id }}&order_id=' + $(element).attr('href'),
                        dataType: 'json',
                        beforeSend: function() {
                                $(element).parent().parent().parent().find('button').button('loading');
                        },
                        complete: function() {
                                $(element).parent().parent().parent().find('button').button('reset');
                        },
                        success: function(json) {
                                $('.alert-dismissible').remove();

                                if (json['error']) {
                                        $('#content > .container-fluid').prepend('<div class="alert alert-danger alert-dismissible"><i class="fa fa-exclamation-circle"></i> ' + json['error'] + ' <button type="button" class="close" data-dismiss="alert">&times;</button></div>');
                                }

                                if (json['success']) {
                                        location = '{{ delete }}';
                                }
                        },
                        error: function(xhr, ajaxOptions, thrownError) {
                                alert(thrownError + "\r\n" + xhr.statusText + "\r\n" + xhr.responseText);
                        }
                });
        }
});
//--></script>
<script src="view/javascript/jquery/datetimepicker/bootstrap-datetimepicker.min.js" type="text/javascript"></script>
<link href="view/javascript/jquery/datetimepicker/bootstrap-datetimepicker.min.css" type="text/css" rel="stylesheet" media="screen"/>
<script type="text/javascript"><!--
$('.date').datetimepicker({
        language: '{{ datepicker }}',
        pickTime: false
});
//--></script>
{{ footer }}
```

admin/view/template/sale/order_info_invoice.twig

```
{{ header }}{{ column_left }}
<div id="content">
  <div class="page-header">
    <div class="container-fluid">
      <div class="pull-right"><a href="{{ invoice }}" target="_blank" data-toggle="tooltip" title="{{ button_invoice_print }}" class="btn btn-info"><i class="fa fa-print"></i></a> <a href="{{ shipping }}" target="_blank" data-toggle="tooltip" title="{{ button_shipping_print }}" class="btn btn-info"><i class="fa fa-truck"></i></a> <a href="{{ cancel }}" data-toggle="tooltip" title="{{ button_cancel }}" class="btn btn-default"><i class="fa fa-reply"></i></a></div>
      <h1>{{ heading_title }}</h1>
      <ul class="breadcrumb">
        {% for breadcrumb in breadcrumbs %}
          <li><a href="{{ breadcrumb.href }}">{{ breadcrumb.text }}</a></li>
        {% endfor %}
      </ul>
    </div>
  </div>
  <div class="container-fluid">
    <div class="row">
      <div class="col-md-4">
        <div class="panel panel-default">
          <div class="panel-heading">
            <h3 class="panel-title"><i class="fa fa-shopping-cart"></i> {{ text_order_detail }}</h3>
          </div>
          <table class="table">
            <tbody>
              <tr>
                <td style="width: 1%;"><button data-toggle="tooltip" title="{{ text_store }}" class="btn btn-info btn-xs"><i class="fa fa-shopping-cart fa-fw"></i></button></td>
                <td><a href="{{ store_url }}" target="_blank">{{ store_name }}</a></td>
              </tr>
              <tr>
                <td><button data-toggle="tooltip" title="{{ text_date_added }}" class="btn btn-info btn-xs"><i class="fa fa-calendar fa-fw"></i></button></td>
                <td>{{ date_added }}</td>
              </tr>
              <tr>
                <td><button data-toggle="tooltip" title="{{ text_payment_method }}" class="btn btn-info btn-xs"><i class="fa fa-credit-card fa-fw"></i></button></td>
                <td>{{ payment_method }}</td>
              </tr>
              {% if shipping_method %}
                <tr>
                  <td><button data-toggle="tooltip" title="{{ text_shipping_method }}" class="btn btn-info btn-xs"><i class="fa fa-truck fa-fw"></i></button></td>
                  <td>{{ shipping_method }}</td>
                </tr>
              {% endif %}
            </tbody>
          </table>
        </div>
      </div>
      <div class="col-md-4">
        <div class="panel panel-default">
          <div class="panel-heading">
            <h3 class="panel-title"><i class="fa fa-user"></i> {{ text_customer_detail }}</h3>
          </div>
          <table class="table">
            <tr>
              <td style="width: 1%;"><button data-toggle="tooltip" title="{{ text_customer }}" class="btn btn-info btn-xs"><i class="fa fa-user fa-fw"></i></button></td>
              <td>{% if customer %} <a href="{{ customer }}" target="_blank">{{ fullname }}</a> {% else %}
                  {{ fullname }}
                {% endif %}</td>
            </tr>
            <tr>
              <td><button data-toggle="tooltip" title="{{ text_customer_group }}" class="btn btn-info btn-xs"><i class="fa fa-group fa-fw"></i></button></td>
              <td>{{ customer_group }}</td>
            </tr>
            <tr>
              <td><button data-toggle="tooltip" title="{{ text_email }}" class="btn btn-info btn-xs"><i class="fa fa-envelope-o fa-fw"></i></button></td>
              <td><a href="mailto:{{ email }}">{{ email }}</a></td>
            </tr>
            <tr>
              <td><button data-toggle="tooltip" title="{{ text_telephone }}" class="btn btn-info btn-xs"><i class="fa fa-phone fa-fw"></i></button></td>
              <td>{{ telephone }}</td>
            </tr>
          </table>
        </div>
      </div>
      <div class="col-md-4">
        <div class="panel panel-default">
          <div class="panel-heading">
            <h3 class="panel-title"><i class="fa fa-cog"></i> {{ text_option }}</h3>
          </div>
          <table class="table">
            <tbody>
              <tr>
                <td>{{ text_invoice }}</td>
                <td id="invoice" class="text-right"><input type="text"  id="get-invoice" name="invoice" value="{{ invoice_no }}" /></td>
                <td style="width: 1%;" class="text-center">
                    <button id="button-invoice" data-loading-text="{{ text_loading }}" data-toggle="tooltip" title="{{ button_save }}" class="btn btn-success btn-xs"><i class="fa fa-cog"></i></button>
<!--
                    <button disabled="disabled" class="btn btn-success btn-xs"><i class="fa fa-refresh"></i></button>
 -->
              </tr>
              <tr>
                <td>{{ text_reward }}</td>
                <td class="text-right">{{ reward }}</td>
                <td class="text-center">{% if customer and reward %}
                    {% if not reward_total %}
                      <button id="button-reward-add" data-loading-text="{{ text_loading }}" data-toggle="tooltip" title="{{ button_reward_add }}" class="btn btn-success btn-xs"><i class="fa fa-plus-circle"></i></button>
                    {% else %}
                      <button id="button-reward-remove" data-loading-text="{{ text_loading }}" data-toggle="tooltip" title="{{ button_reward_remove }}" class="btn btn-danger btn-xs"><i class="fa fa-minus-circle"></i></button>
                    {% endif %}
                  {% else %}
                    <button disabled="disabled" class="btn btn-success btn-xs"><i class="fa fa-plus-circle"></i></button>
                  {% endif %}</td>
              </tr>
              <tr>
                <td>{{ text_affiliate }}
                  {% if affiliate %}
                    (<a href="{{ affiliate }}">{{ affiliate_fullname }}</a>)
                  {% endif %}</td>
                <td class="text-right">{{ commission }}</td>
                <td class="text-center">{% if affiliate %}
                    {% if not commission_total %}
                      <button id="button-commission-add" data-loading-text="{{ text_loading }}" data-toggle="tooltip" title="{{ button_commission_add }}" class="btn btn-success btn-xs"><i class="fa fa-plus-circle"></i></button>
                    {% else %}
                      <button id="button-commission-remove" data-loading-text="{{ text_loading }}" data-toggle="tooltip" title="{{ button_commission_remove }}" class="btn btn-danger btn-xs"><i class="fa fa-minus-circle"></i></button>
                    {% endif %}
                  {% else %}
                    <button disabled="disabled" class="btn btn-success btn-xs"><i class="fa fa-plus-circle"></i></button>
                  {% endif %}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <div class="panel panel-default">
      <div class="panel-heading">
        <h3 class="panel-title"><i class="fa fa-info-circle"></i> {{ text_order }}</h3>
      </div>
      <div class="panel-body">
        {% if shipping_method %}
        <table class="table table-bordered">
          <thead>
            <tr>
              <td style="width: 50%;" class="text-left">{{ text_shipping_address }}</td>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td class="text-left">{{ shipping_address }}</td>
            </tr>
          </tbody>
        </table>
        {% endif %}
        <table class="table table-bordered">
          <thead>
            <tr>
              <td class="text-left">{{ column_product }}</td>
              <td class="text-left">{{ column_model }}</td>
              <td class="text-right">{{ column_quantity }}</td>
              <td class="text-right">{{ column_price }}</td>
              <td class="text-right">{{ column_total }}</td>
            </tr>
          </thead>
          <tbody>
            {% for product in products %}
              <tr>
                <td class="text-left"><a href="{{ product.href }}">{{ product.name }}</a>
                  {% for option in product.option %}
                    <br/>
                    {% if option.type != 'file' %}&nbsp;
                      <small> - {{ option.name }}: {{ option.value }}</small>
                      {% else %}
                      &nsbr
                      <small> - {{ option.name }}: <a href="{{ option.href }}">{{ option.value }}</a></small>
                    {% endif %}
                  {% endfor %}</td>
                <td class="text-left">{{ product.model }}</td>
                <td class="text-right">{{ product.quantity }}</td>
                <td class="text-right">{{ product.price }}</td>
                <td class="text-right">{{ product.total }}</td>
              </tr>
            {% endfor %}
            {% for voucher in vouchers %}
              <tr>
                <td class="text-left"><a href="{{ voucher.href }}">{{ voucher.description }}</a></td>
                <td class="text-left"></td>
                <td class="text-right">1</td>
                <td class="text-right">{{ voucher.amount }}</td>
                <td class="text-right">{{ voucher.amount }}</td>
              </tr>
            {% endfor %}
            {% for total in totals %}
              <tr>
                <td colspan="4" class="text-right">{{ total.title }}</td>
                <td class="text-right">{{ total.text }}</td>
              </tr>
            {% endfor %}
          </tbody>
        </table>
        {% if comment %}
          <table class="table table-bordered">
            <thead>
              <tr>
                <td>{{ text_comment }}</td>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>{{ comment }}</td>
              </tr>
            </tbody>
          </table>
        {% endif %}</div>
    </div>
    <div class="panel panel-default">
      <div class="panel-heading">
        <h3 class="panel-title"><i class="fa fa-comment-o"></i> {{ text_history }}</h3>
      </div>
      <div class="panel-body">
        <ul class="nav nav-tabs">
          <li class="active"><a href="#tab-history" data-toggle="tab">{{ tab_history }}</a></li>
          <li><a href="#tab-additional" data-toggle="tab">{{ tab_additional }}</a></li>
          {% for tab in tabs %}
            <li><a href="#tab-{{ tab.code }}" data-toggle="tab">{{ tab.title }}</a></li>
          {% endfor %}
        </ul>
        <div class="tab-content">
         <div class="tab-pane active" id="tab-history">
           <div id="history"></div>
           <br/>
         </div>
          <div class="tab-pane" id="tab-additional"> {% if account_custom_fields %}
              <div class="table-responsive">
                <table class="table table-bordered">
                  <thead>
                    <tr>
                      <td colspan="2">{{ text_account_custom_field }}</td>
                    </tr>
                  </thead>
                  <tbody>
                    {% for custom_field in account_custom_fields %}
                      <tr>
                        <td>{{ custom_field.name }}</td>
                        <td>{{ custom_field.value }}</td>
                      </tr>
                    {% endfor %}
                  </tbody>
                </table>
              </div>
            {% endif %}
            {% if payment_custom_fields %}
              <div class="table-responsive">
                <table class="table table-bordered">
                  <thead>
                    <tr>
                      <td colspan="2">{{ text_payment_custom_field }}</td>
                    </tr>
                  </thead>
                  <tbody>
                    {% for custom_field in payment_custom_fields %}
                      <tr>
                        <td>{{ custom_field.name }}</td>
                        <td>{{ custom_field.value }}</td>
                      </tr>
                    {% endfor %}
                  </tbody>
                </table>
              </div>
            {% endif %}
            {% if shipping_method and shipping_custom_fields %}
              <div class="table-responsive">
                <table class="table table-bordered">
                  <thead>
                    <tr>
                      <td colspan="2">{{ text_shipping_custom_field }}</td>
                    </tr>
                  </thead>
                  <tbody>
                    {% for custom_field in shipping_custom_fields %}
                      <tr>
                        <td>{{ custom_field.name }}</td>
                        <td>{{ custom_field.value }}</td>
                      </tr>
                    {% endfor %}
                  </tbody>
                </table>
              </div>
            {% endif %}
            <div class="table-responsive">
              <table class="table table-bordered">
                <thead>
                  <tr>
                    <td colspan="2">{{ text_browser }}</td>
                  </tr>
                </thead>
                <tbody>
                  <tr>
                    <td>{{ text_ip }}</td>
                    <td>{{ ip }}</td>
                  </tr>
                  {% if forwarded_ip %}
                    <tr>
                      <td>{{ text_forwarded_ip }}</td>
                      <td>{{ forwarded_ip }}</td>
                    </tr>
                  {% endif %}
                  <tr>
                    <td>{{ text_user_agent }}</td>
                    <td>{{ user_agent }}</td>
                  </tr>
                  <tr>
                    <td>{{ text_accept_language }}</td>
                    <td>{{ accept_language }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
          {% for tab in tabs %}
            <div class="tab-pane" id="tab-{{ tab.code }}">{{ tab.content }}</div>
          {% endfor %}</div>
      </div>
    </div>
  </div>
</div>
<script type="text/javascript"><!--
$(document).delegate('#button-invoice', 'click', function() {
        alert("發票號碼 : "+document.getElementById('get-invoice').value);
        $.ajax({
                url: 'index.php?route=sale/invoice/createinvoiceno&user_token={{ user_token }}&order_id={{ order_id }}&InvoiceNo='+document.getElementById('get-invoice').value,
                dataType: 'json',
                beforeSend: function() {
                        $('#button-invoice').button('loading');
                },
                complete: function() {
                        $('#button-invoice').button('reset');
                },
                success: function(json) {
                        $('.alert-dismissible').remove();

                        if (json['error']) {
                                $('#content > .container-fluid').prepend('<div class="alert alert-danger alert-dismissible"><i class="fa fa-exclamation-circle"></i> ' + json['error'] + '</div>');
                        }

                        if (json['invoice_no']) {
                                $('#invoice').html(json['invoice_no']);

                                $('#button-invoice').replaceWith('<button disabled="disabled" class="btn btn-success btn-xs"><i class="fa fa-cog"></i></button>');
                        }
                },
                error: function(xhr, ajaxOptions, thrownError) {
                        alert(thrownError + "\r\n" + xhr.statusText + "\r\n" + xhr.responseText);
                }
        });
});

$(document).delegate('#button-reward-add', 'click', function() {
        $.ajax({
                url: 'index.php?route=sale/invoice/addreward&user_token={{ user_token }}&order_id={{ order_id }}',
                type: 'post',
                dataType: 'json',
                beforeSend: function() {
                        $('#button-reward-add').button('loading');
                },
                complete: function() {
                        $('#button-reward-add').button('reset');
                },
                success: function(json) {
                        $('.alert-dismissible').remove();

                        if (json['error']) {
                                $('#content > .container-fluid').prepend('<div class="alert alert-danger alert-dismissible"><i class="fa fa-exclamation-circle"></i> ' + json['error'] + '</div>');
                        }

                        if (json['success']) {
                                $('#content > .container-fluid').prepend('<div class="alert alert-success alert-dismissible"><i class="fa fa-check-circle"></i> ' + json['success'] + '</div>');

                                $('#button-reward-add').replaceWith('<button id="button-reward-remove" data-toggle="tooltip" title="{{ button_reward_remove }}" class="btn btn-danger btn-xs"><i class="fa fa-minus-circle"></i></button>');
                        }
                },
                error: function(xhr, ajaxOptions, thrownError) {
                        alert(thrownError + "\r\n" + xhr.statusText + "\r\n" + xhr.responseText);
                }
        });
});

$(document).delegate('#button-reward-remove', 'click', function() {
        $.ajax({
                url: 'index.php?route=sale/invoice/removereward&user_token={{ user_token }}&order_id={{ order_id }}',
                type: 'post',
                dataType: 'json',
                beforeSend: function() {
                        $('#button-reward-remove').button('loading');
                },
                complete: function() {
                        $('#button-reward-remove').button('reset');
                },
                success: function(json) {
                        $('.alert-dismissible').remove();

                        if (json['error']) {
                                $('#content > .container-fluid').prepend('<div class="alert alert-danger alert-dismissible"><i class="fa fa-exclamation-circle"></i> ' + json['error'] + '</div>');
                        }

                        if (json['success']) {
                                $('#content > .container-fluid').prepend('<div class="alert alert-success alert-dismissible"><i class="fa fa-check-circle"></i> ' + json['success'] + '</div>');

                                $('#button-reward-remove').replaceWith('<button id="button-reward-add" data-toggle="tooltip" title="{{ button_reward_add }}" class="btn btn-success btn-xs"><i class="fa fa-plus-circle"></i></button>');
                        }
                },
                error: function(xhr, ajaxOptions, thrownError) {
                        alert(thrownError + "\r\n" + xhr.statusText + "\r\n" + xhr.responseText);
                }
        });
});

$(document).delegate('#button-commission-add', 'click', function() {
        $.ajax({
                url: 'index.php?route=sale/invoice/addcommission&user_token={{ user_token }}&order_id={{ order_id }}',
                type: 'post',
                dataType: 'json',
                beforeSend: function() {
                        $('#button-commission-add').button('loading');
                },
                complete: function() {
                        $('#button-commission-add').button('reset');
                },
                success: function(json) {
                        $('.alert-dismissible').remove();

                        if (json['error']) {
                                $('#content > .container-fluid').prepend('<div class="alert alert-danger alert-dismissible"><i class="fa fa-exclamation-circle"></i> ' + json['error'] + '</div>');
                        }

                        if (json['success']) {
                                $('#content > .container-fluid').prepend('<div class="alert alert-success alert-dismissible"><i class="fa fa-check-circle"></i> ' + json['success'] + '</div>');

                                $('#button-commission-add').replaceWith('<button id="button-commission-remove" data-toggle="tooltip" title="{{ button_commission_remove }}" class="btn btn-danger btn-xs"><i class="fa fa-minus-circle"></i></button>');
                        }
                },
                error: function(xhr, ajaxOptions, thrownError) {
                        alert(thrownError + "\r\n" + xhr.statusText + "\r\n" + xhr.responseText);
                }
        });
});

$(document).delegate('#button-commission-remove', 'click', function() {
        $.ajax({
                url: 'index.php?route=sale/invoice/removecommission&user_token={{ user_token }}&order_id={{ order_id }}',
                type: 'post',
                dataType: 'json',
                beforeSend: function() {
                        $('#button-commission-remove').button('loading');
                },
                complete: function() {
                        $('#button-commission-remove').button('reset');
                },
                success: function(json) {
                        $('.alert-dismissible').remove();

                        if (json['error']) {
                                $('#content > .container-fluid').prepend('<div class="alert alert-danger alert-dismissible"><i class="fa fa-exclamation-circle"></i> ' + json['error'] + '</div>');
                        }

                        if (json['success']) {
                                $('#content > .container-fluid').prepend('<div class="alert alert-success alert-dismissible"><i class="fa fa-check-circle"></i> ' + json['success'] + '</div>');

                                $('#button-commission-remove').replaceWith('<button id="button-commission-add" data-toggle="tooltip" title="{{ button_commission_add }}" class="btn btn-success btn-xs"><i class="fa fa-plus-circle"></i></button>');
                        }
                },
                error: function(xhr, ajaxOptions, thrownError) {
                        alert(thrownError + "\r\n" + xhr.statusText + "\r\n" + xhr.responseText);
                }
        });
});

$('#history').delegate('.pagination a', 'click', function(e) {
        e.preventDefault();

        $('#history').load(this.href);
});

$('#history').load('index.php?route=sale/order/history&user_token={{ user_token }}&order_id={{ order_id }}');

$('#button-history').on('click', function() {
        $.ajax({
                url: '{{ catalog }}index.php?route=api/invoice/history&api_token={{ api_token }}&store_id={{ store_id }}&order_id={{ order_id }}',
                type: 'post',
                dataType: 'json',
                data: 'order_status_id=' + encodeURIComponent($('select[name=\'order_status_id\']').val()) + '&notify=' + ($('input[name=\'notify\']').prop('checked') ? 1 : 0) + '&override=' + ($('input[name=\'override\']').prop('checked') ? 1 : 0) + '&append=' + ($('input[name=\'append\']').prop('checked') ? 1 : 0) + '&comment=' + encodeURIComponent($('textarea[name=\'comment\']').val()),
                beforeSend: function() {
                        $('#button-history').button('loading');
                },
                complete: function() {
                        $('#button-history').button('reset');
                },
                success: function(json) {
                        $('.alert-dismissible').remove();

                        if (json['error']) {
                                $('#history').before('<div class="alert alert-danger alert-dismissible"><i class="fa fa-exclamation-circle"></i> ' + json['error'] + ' <button type="button" class="close" data-dismiss="alert">&times;</button></div>');
                        }

                        if (json['success']) {
                                $('#history').load('index.php?route=sale/order/history&user_token={{ user_token }}&order_id={{ order_id }}');

                                $('#history').before('<div class="alert alert-success alert-dismissible"><i class="fa fa-check-circle"></i> ' + json['success'] + ' <button type="button" class="close" data-dismiss="alert">&times;</button></div>');

                                $('textarea[name=\'comment\']').val('');
                        }
                },
                error: function(xhr, ajaxOptions, thrownError) {
                        alert(thrownError + "\r\n" + xhr.statusText + "\r\n" + xhr.responseText);
                }
        });
});
//--></script>
{{ footer }}
```

/admin/model/sale/invoice.php

```
<?php
class ModelSaleInvoice extends Model {
        public function getOrder($order_id) {
                $order_query = $this->db->query("SELECT *, (SELECT c.fullname FROM " . DB_PREFIX . "customer c WHERE c.customer_id = o.customer_id) AS customer, (SELECT os.name FROM " . DB_PREFIX . "order_status os WHERE os.order_status_id = o.order_status_id AND os.language_id = '" . (int)$this->config->get('config_language_id') . "') AS order_status FROM `" . DB_PREFIX . "order` o WHERE o.order_id = '" . (int)$order_id . "'");

                if ($order_query->num_rows) {
                        $country_query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "country` WHERE country_id = '" . (int)$order_query->row['payment_country_id'] . "'");

                        if ($country_query->num_rows) {
                                $payment_iso_code_2 = $country_query->row['iso_code_2'];
                                $payment_iso_code_3 = $country_query->row['iso_code_3'];
                        } else {
                                $payment_iso_code_2 = '';
                                $payment_iso_code_3 = '';
                        }

                        $zone_query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "zone` WHERE zone_id = '" . (int)$order_query->row['payment_zone_id'] . "'");

                        if ($zone_query->num_rows) {
                                $payment_zone_code = $zone_query->row['code'];
                        } else {
                                $payment_zone_code = '';
                        }

                        $country_query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "country` WHERE country_id = '" . (int)$order_query->row['shipping_country_id'] . "'");

                        if ($country_query->num_rows) {
                                $shipping_iso_code_2 = $country_query->row['iso_code_2'];
                                $shipping_iso_code_3 = $country_query->row['iso_code_3'];
                        } else {
                                $shipping_iso_code_2 = '';
                                $shipping_iso_code_3 = '';
                        }

                        $zone_query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "zone` WHERE zone_id = '" . (int)$order_query->row['shipping_zone_id'] . "'");

                        if ($zone_query->num_rows) {
                                $shipping_zone_code = $zone_query->row['code'];
                        } else {
                                $shipping_zone_code = '';
                        }

                        $reward = 0;

                        $order_product_query = $this->db->query("SELECT * FROM " . DB_PREFIX . "order_product WHERE order_id = '" . (int)$order_id . "'");

                        foreach ($order_product_query->rows as $product) {
                                $reward += $product['reward'];
                        }

                        $this->load->model('customer/customer');

                        $affiliate_info = $this->model_customer_customer->getCustomer($order_query->row['affiliate_id']);

                        if ($affiliate_info) {
                                $affiliate_fullname = $affiliate_info['fullname'];
                        } else {
                                $affiliate_fullname = '';
                        }

                        $this->load->model('localisation/language');

                        $language_info = $this->model_localisation_language->getLanguage($order_query->row['language_id']);

                        if ($language_info) {
                                $language_code = $language_info['code'];
                        } else {
                                $language_code = $this->config->get('config_language');
                        }

                        return array(
                                'order_id'                => $order_query->row['order_id'],
                                'invoice_no'              => $order_query->row['invoice_no'],
                                'invoice_prefix'          => $order_query->row['invoice_prefix'],
                                'store_id'                => $order_query->row['store_id'],
                                'store_name'              => $order_query->row['store_name'],
                                'store_url'               => $order_query->row['store_url'],
                                'customer_id'             => $order_query->row['customer_id'],
                                'customer'                => $order_query->row['customer'],
                                'customer_group_id'       => $order_query->row['customer_group_id'],
                                'fullname'               => $order_query->row['fullname'],
                                'email'                   => $order_query->row['email'],
                                'telephone'               => $order_query->row['telephone'],
                                'custom_field'            => json_decode($order_query->row['custom_field'], true),
                                'payment_fullname'       => $order_query->row['payment_fullname'],
                'payment_telephone'       => $order_query->row['payment_telephone'],
                                'payment_company'         => $order_query->row['payment_company'],
                                'payment_address_1'       => $order_query->row['payment_address_1'],
                                'payment_address_2'       => $order_query->row['payment_address_2'],
                                'payment_postcode'        => $order_query->row['payment_postcode'],
                'payment_city_id'         => $order_query->row['payment_city_id'],
                                'payment_city'            => $order_query->row['payment_city'],
                                'payment_zone_id'         => $order_query->row['payment_zone_id'],
                                'payment_zone'            => $order_query->row['payment_zone'],
                                'payment_zone_code'       => $payment_zone_code,
                                'payment_country_id'      => $order_query->row['payment_country_id'],
                                'payment_country'         => $order_query->row['payment_country'],
                'payment_county_id'      => $order_query->row['payment_county_id'],
                'payment_county'         => $order_query->row['payment_county'],
                                'payment_iso_code_2'      => $payment_iso_code_2,
                                'payment_iso_code_3'      => $payment_iso_code_3,
                                'payment_address_format'  => $order_query->row['payment_address_format'],
                                'payment_custom_field'    => json_decode($order_query->row['payment_custom_field'], true),
                                'payment_method'          => $order_query->row['payment_method'],
                                'payment_code'            => $order_query->row['payment_code'],
                                'shipping_fullname'      => $order_query->row['shipping_fullname'],
                'shipping_telephone'      => $order_query->row['shipping_telephone'],
                                'shipping_company'        => $order_query->row['shipping_company'],
                                'shipping_address_1'      => $order_query->row['shipping_address_1'],
                                'shipping_address_2'      => $order_query->row['shipping_address_2'],
                                'shipping_postcode'       => $order_query->row['shipping_postcode'],
                'shipping_city_id'        => $order_query->row['shipping_city_id'],
                                'shipping_city'           => $order_query->row['shipping_city'],
                                'shipping_zone_id'        => $order_query->row['shipping_zone_id'],
                                'shipping_zone'           => $order_query->row['shipping_zone'],
                                'shipping_zone_code'      => $shipping_zone_code,
                                'shipping_country_id'     => $order_query->row['shipping_country_id'],
                                'shipping_country'        => $order_query->row['shipping_country'],
                'shipping_county_id'      => $order_query->row['shipping_county_id'],
                'shipping_county'         => $order_query->row['shipping_county'],
                                'shipping_iso_code_2'     => $shipping_iso_code_2,
                                'shipping_iso_code_3'     => $shipping_iso_code_3,
                                'shipping_address_format' => $order_query->row['shipping_address_format'],
                                'shipping_custom_field'   => json_decode($order_query->row['shipping_custom_field'], true),
                                'shipping_method'         => $order_query->row['shipping_method'],
                                'shipping_code'           => $order_query->row['shipping_code'],
                                'comment'                 => $order_query->row['comment'],
                                'total'                   => $order_query->row['total'],
                                'reward'                  => $reward,
                                'order_status_id'         => $order_query->row['order_status_id'],
                                'order_status'            => $order_query->row['order_status'],
                                'affiliate_id'            => $order_query->row['affiliate_id'],
                                'affiliate_fullname'     => $affiliate_fullname,
                                'commission'              => $order_query->row['commission'],
                                'language_id'             => $order_query->row['language_id'],
                                'language_code'           => $language_code,
                                'currency_id'             => $order_query->row['currency_id'],
                                'currency_code'           => $order_query->row['currency_code'],
                                'currency_value'          => $order_query->row['currency_value'],
                                'ip'                      => $order_query->row['ip'],
                                'forwarded_ip'            => $order_query->row['forwarded_ip'],
                                'user_agent'              => $order_query->row['user_agent'],
                                'accept_language'         => $order_query->row['accept_language'],
                                'date_added'              => $order_query->row['date_added'],
                        );
                } else {
                        return;
                }
        }

        public function getOrders($data = array()) {
                $sql = "SELECT o.order_id, o.fullname AS customer, (SELECT os.name FROM " . DB_PREFIX . "order_status os WHERE os.order_status_id = o.order_status_id AND os.language_id = '" . (int)$this->config->get('config_language_id') . "') AS order_status, o.shipping_code, o.total, o.currency_code, o.currency_value, o.date_added, o.invoice_no FROM `" . DB_PREFIX . "order` o";

                if (!empty($data['filter_order_status'])) {
                        $implode = array();

                        $order_statuses = explode(',', $data['filter_order_status']);

                        foreach ($order_statuses as $order_status_id) {
                                $implode[] = "o.order_status_id = '" . (int)$order_status_id . "'";
                        }

                        if ($implode) {
                                $sql .= " WHERE (" . implode(" OR ", $implode) . ")";
                        }
                } elseif (isset($data['filter_order_status_id']) && $data['filter_order_status_id'] !== '') {
                        $sql .= " WHERE o.order_status_id = '" . (int)$data['filter_order_status_id'] . "'";
                } else {
                        $sql .= " WHERE o.order_status_id > '0'";
                }

                if (!empty($data['filter_order_id'])) {
                        $sql .= " AND o.order_id = '" . (int)$data['filter_order_id'] . "'";
                }

                if (!empty($data['filter_customer'])) {
                        $sql .= " AND o.fullname LIKE '%" . $this->db->escape((string)$data['filter_customer']) . "%'";
                }

                if (!empty($data['filter_date_added'])) {
                        $sql .= " AND DATE(o.date_added) >= DATE('" . $this->db->escape((string)$data['filter_date_added']) . "')";
                }


        if (!empty($data['filter_total'])) {
            if (stripos($data['filter_total'], '-')) {
                $totals = explode('-', $data['filter_total']);
                $from_total = $totals[0];
                $to_total = $totals[1];
                $sql .= " AND o.total >= '" . (float)$from_total . "' AND o.total <= '" . (float)$to_total . "'";
            } else {
                $sql .= " AND o.total = '" . (float)$data['filter_total'] . "'";
            }
        }

                $sort_data = array(
                        'o.order_id',
                        'customer',
                        'order_status',
                        'o.date_added',
                        'o.invoice_no',
                        'o.total'
                );

                if (isset($data['sort']) && in_array($data['sort'], $sort_data)) {
                        $sql .= " ORDER BY " . $data['sort'];
                } else {
                        $sql .= " ORDER BY o.order_id";
                }

                if (isset($data['order']) && ($data['order'] == 'DESC')) {
                        $sql .= " DESC";
                } else {
                        $sql .= " ASC";
                }

                if (isset($data['start']) || isset($data['limit'])) {
                        if ($data['start'] < 0) {
                                $data['start'] = 0;
                        }

                        if ($data['limit'] < 1) {
                                $data['limit'] = 20;
                        }

                        $sql .= " LIMIT " . (int)$data['start'] . "," . (int)$data['limit'];
                }

                $query = $this->db->query($sql);

                return $query->rows;
        }

        public function getOrderProducts($order_id) {
                $query = $this->db->query("SELECT * FROM " . DB_PREFIX . "order_product WHERE order_id = '" . (int)$order_id . "'");

                return $query->rows;
        }

        public function getOrderOptions($order_id, $order_product_id) {
                $query = $this->db->query("SELECT * FROM " . DB_PREFIX . "order_option WHERE order_id = '" . (int)$order_id . "' AND order_product_id = '" . (int)$order_product_id . "'");

                return $query->rows;
        }

        public function getOrderVouchers($order_id) {
                $query = $this->db->query("SELECT * FROM " . DB_PREFIX . "order_voucher WHERE order_id = '" . (int)$order_id . "'");

                return $query->rows;
        }

        public function getOrderVoucherByVoucherId($voucher_id) {
                $query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "order_voucher` WHERE voucher_id = '" . (int)$voucher_id . "'");

                return $query->row;
        }

        public function getOrderTotals($order_id) {
                $query = $this->db->query("SELECT * FROM " . DB_PREFIX . "order_total WHERE order_id = '" . (int)$order_id . "' ORDER BY sort_order");

                return $query->rows;
        }

        public function getTotalOrders($data = array()) {
                $sql = "SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order`";

                if (!empty($data['filter_order_status'])) {
                        $implode = array();

                        $order_statuses = explode(',', $data['filter_order_status']);

                        foreach ($order_statuses as $order_status_id) {
                                $implode[] = "order_status_id = '" . (int)$order_status_id . "'";
                        }

                        if ($implode) {
                                $sql .= " WHERE (" . implode(" OR ", $implode) . ")";
                        }
                } elseif (isset($data['filter_order_status_id']) && $data['filter_order_status_id'] !== '') {
                        $sql .= " WHERE order_status_id = '" . (int)$data['filter_order_status_id'] . "'";
                } else {
                        $sql .= " WHERE order_status_id > '0'";
                }

                if (!empty($data['filter_order_id'])) {
                        $sql .= " AND order_id = '" . (int)$data['filter_order_id'] . "'";
                }

                if (!empty($data['filter_customer'])) {
                        $sql .= " AND fullname LIKE '%" . $this->db->escape((string)$data['filter_customer']) . "%'";
                }

                if (!empty($data['filter_date_added'])) {
                        $sql .= " AND DATE(date_added) >= DATE('" . $this->db->escape((string)$data['filter_date_added']) . "')";
                }

        if (!empty($data['filter_total'])) {
            if (stripos($data['filter_total'], '-')) {
                $totals = explode('-', $data['filter_total']);
                $from_total = $totals[0];
                $to_total = $totals[1];
                $sql .= " AND total >= '" . (float)$from_total . "' AND total <= '" . (float)$to_total . "'";
            } else {
                $sql .= " AND total = '" . (float)$data['filter_total'] . "'";
            }
        }

                $query = $this->db->query($sql);

                return $query->row['total'];
        }

        public function getTotalOrdersByStoreId($store_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE store_id = '" . (int)$store_id . "'");

                return $query->row['total'];
        }

        public function getTotalOrdersByOrderStatusId($order_status_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE order_status_id = '" . (int)$order_status_id . "' AND order_status_id > '0'");

                return $query->row['total'];
        }

        public function getTotalOrdersByProcessingStatus() {
                $implode = array();

                $order_statuses = $this->config->get('config_processing_status');

                foreach ($order_statuses as $order_status_id) {
                        $implode[] = "order_status_id = '" . (int)$order_status_id . "'";
                }

                if ($implode) {
                        $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE " . implode(" OR ", $implode));

                        return $query->row['total'];
                } else {
                        return 0;
                }
        }

        public function getTotalOrdersByCompleteStatus() {
                $implode = array();

                $order_statuses = $this->config->get('config_complete_status');

                foreach ($order_statuses as $order_status_id) {
                        $implode[] = "order_status_id = '" . (int)$order_status_id . "'";
                }

                if ($implode) {
                        $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE " . implode(" OR ", $implode) . "");

                        return $query->row['total'];
                } else {
                        return 0;
                }
        }

        public function getTotalOrdersByLanguageId($language_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE language_id = '" . (int)$language_id . "' AND order_status_id > '0'");

                return $query->row['total'];
        }

        public function getTotalOrdersByCurrencyId($currency_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE currency_id = '" . (int)$currency_id . "' AND order_status_id > '0'");

                return $query->row['total'];
        }

        public function getTotalSales($data = array()) {
                $sql = "SELECT SUM(total) AS total FROM `" . DB_PREFIX . "order`";

                if (!empty($data['filter_order_status'])) {
                        $implode = array();

                        $order_statuses = explode(',', $data['filter_order_status']);

                        foreach ($order_statuses as $order_status_id) {
                                $implode[] = "order_status_id = '" . (int)$order_status_id . "'";
                        }

                        if ($implode) {
                                $sql .= " WHERE (" . implode(" OR ", $implode) . ")";
                        }
                } elseif (isset($data['filter_order_status_id']) && $data['filter_order_status_id'] !== '') {
                        $sql .= " WHERE order_status_id = '" . (int)$data['filter_order_status_id'] . "'";
                } else {
                        $sql .= " WHERE order_status_id > '0'";
                }

                if (!empty($data['filter_order_id'])) {
                        $sql .= " AND order_id = '" . (int)$data['filter_order_id'] . "'";
                }

                if (!empty($data['filter_customer'])) {
                        $sql .= " AND fullname LIKE '%" . $this->db->escape((string)$data['filter_customer']) . "%'";
                }

                if (!empty($data['filter_date_added'])) {
                        $sql .= " AND DATE(date_added) = DATE('" . $this->db->escape((string)$data['filter_date_added']) . "')";
                }

                if (!empty($data['filter_total'])) {
                        $sql .= " AND total = '" . (float)$data['filter_total'] . "'";
                }

                $query = $this->db->query($sql);

                return $query->row['total'];
        }

        public function getOrderHistories($order_id, $start = 0, $limit = 10) {
                if ($start < 0) {
                        $start = 0;
                }

                if ($limit < 1) {
                        $limit = 10;
                }

                $query = $this->db->query("SELECT oh.date_added, os.name AS status, oh.comment, oh.notify FROM " . DB_PREFIX . "order_history oh LEFT JOIN " . DB_PREFIX . "order_status os ON oh.order_status_id = os.order_status_id WHERE oh.order_id = '" . (int)$order_id . "' AND os.language_id = '" . (int)$this->config->get('config_language_id') . "' ORDER BY oh.date_added DESC LIMIT " . (int)$start . "," . (int)$limit);

                return $query->rows;
        }

        public function getTotalOrderHistories($order_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM " . DB_PREFIX . "order_history WHERE order_id = '" . (int)$order_id . "'");

                return $query->row['total'];
        }

        public function getTotalOrderHistoriesByOrderStatusId($order_status_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM " . DB_PREFIX . "order_history WHERE order_status_id = '" . (int)$order_status_id . "'");

                return $query->row['total'];
        }

        public function getEmailsByProductsOrdered($products, $start, $end) {
                $implode = array();

                foreach ($products as $product_id) {
                        $implode[] = "op.product_id = '" . (int)$product_id . "'";
                }

                $query = $this->db->query("SELECT DISTINCT email FROM `" . DB_PREFIX . "order` o LEFT JOIN " . DB_PREFIX . "order_product op ON (o.order_id = op.order_id) WHERE (" . implode(" OR ", $implode) . ") AND o.order_status_id <> '0' LIMIT " . (int)$start . "," . (int)$end);

                return $query->rows;
        }

        public function getTotalEmailsByProductsOrdered($products) {
                $implode = array();

                foreach ($products as $product_id) {
                        $implode[] = "op.product_id = '" . (int)$product_id . "'";
                }

                $query = $this->db->query("SELECT COUNT(DISTINCT email) AS total FROM `" . DB_PREFIX . "order` o LEFT JOIN " . DB_PREFIX . "order_product op ON (o.order_id = op.order_id) WHERE (" . implode(" OR ", $implode) . ") AND o.order_status_id <> '0'");

                return $query->row['total'];
        }
}
```

admin/model/sale/order.php

```
<?php
class ModelSaleOrder extends Model {
        public function getOrder($order_id) {
                $order_query = $this->db->query("SELECT *, (SELECT c.fullname FROM " . DB_PREFIX . "customer c WHERE c.customer_id = o.customer_id) AS customer, (SELECT os.name FROM " . DB_PREFIX . "order_status os WHERE os.order_status_id = o.order_status_id AND os.language_id = '" . (int)$this->config->get('config_language_id') . "') AS order_status FROM `" . DB_PREFIX . "order` o WHERE o.order_id = '" . (int)$order_id . "'");

                if ($order_query->num_rows) {
                        $country_query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "country` WHERE country_id = '" . (int)$order_query->row['payment_country_id'] . "'");

                        if ($country_query->num_rows) {
                                $payment_iso_code_2 = $country_query->row['iso_code_2'];
                                $payment_iso_code_3 = $country_query->row['iso_code_3'];
                        } else {
                                $payment_iso_code_2 = '';
                                $payment_iso_code_3 = '';
                        }

                        $zone_query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "zone` WHERE zone_id = '" . (int)$order_query->row['payment_zone_id'] . "'");

                        if ($zone_query->num_rows) {
                                $payment_zone_code = $zone_query->row['code'];
                        } else {
                                $payment_zone_code = '';
                        }

                        $country_query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "country` WHERE country_id = '" . (int)$order_query->row['shipping_country_id'] . "'");

                        if ($country_query->num_rows) {
                                $shipping_iso_code_2 = $country_query->row['iso_code_2'];
                                $shipping_iso_code_3 = $country_query->row['iso_code_3'];
                        } else {
                                $shipping_iso_code_2 = '';
                                $shipping_iso_code_3 = '';
                        }

                        $zone_query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "zone` WHERE zone_id = '" . (int)$order_query->row['shipping_zone_id'] . "'");

                        if ($zone_query->num_rows) {
                                $shipping_zone_code = $zone_query->row['code'];
                        } else {
                                $shipping_zone_code = '';
                        }

                        $reward = 0;

                        $order_product_query = $this->db->query("SELECT * FROM " . DB_PREFIX . "order_product WHERE order_id = '" . (int)$order_id . "'");

                        foreach ($order_product_query->rows as $product) {
                                $reward += $product['reward'];
                        }

                        $this->load->model('customer/customer');

                        $affiliate_info = $this->model_customer_customer->getCustomer($order_query->row['affiliate_id']);

                        if ($affiliate_info) {
                                $affiliate_fullname = $affiliate_info['fullname'];
                        } else {
                                $affiliate_fullname = '';
                        }

                        $this->load->model('localisation/language');

                        $language_info = $this->model_localisation_language->getLanguage($order_query->row['language_id']);

                        if ($language_info) {
                                $language_code = $language_info['code'];
                        } else {
                                $language_code = $this->config->get('config_language');
                        }

                        return array(
                                'order_id'                => $order_query->row['order_id'],
                                'invoice_no'              => $order_query->row['invoice_no'],
                                'invoice_prefix'          => $order_query->row['invoice_prefix'],
                                'store_id'                => $order_query->row['store_id'],
                                'store_name'              => $order_query->row['store_name'],
                                'store_url'               => $order_query->row['store_url'],
                                'customer_id'             => $order_query->row['customer_id'],
                                'customer'                => $order_query->row['customer'],
                                'customer_group_id'       => $order_query->row['customer_group_id'],
                                'fullname'               => $order_query->row['fullname'],
                                'email'                   => $order_query->row['email'],
                                'telephone'               => $order_query->row['telephone'],
                                'custom_field'            => json_decode($order_query->row['custom_field'], true),
                                'payment_fullname'       => $order_query->row['payment_fullname'],
                'payment_telephone'       => $order_query->row['payment_telephone'],
                                'payment_company'         => $order_query->row['payment_company'],
                                'payment_address_1'       => $order_query->row['payment_address_1'],
                                'payment_address_2'       => $order_query->row['payment_address_2'],
                                'payment_postcode'        => $order_query->row['payment_postcode'],
                'payment_city_id'         => $order_query->row['payment_city_id'],
                                'payment_city'            => $order_query->row['payment_city'],
                                'payment_zone_id'         => $order_query->row['payment_zone_id'],
                                'payment_zone'            => $order_query->row['payment_zone'],
                                'payment_zone_code'       => $payment_zone_code,
                                'payment_country_id'      => $order_query->row['payment_country_id'],
                                'payment_country'         => $order_query->row['payment_country'],
                'payment_county_id'      => $order_query->row['payment_county_id'],
                'payment_county'         => $order_query->row['payment_county'],
                                'payment_iso_code_2'      => $payment_iso_code_2,
                                'payment_iso_code_3'      => $payment_iso_code_3,
                                'payment_address_format'  => $order_query->row['payment_address_format'],
                                'payment_custom_field'    => json_decode($order_query->row['payment_custom_field'], true),
                                'payment_method'          => $order_query->row['payment_method'],
                                'payment_code'            => $order_query->row['payment_code'],
                                'shipping_fullname'      => $order_query->row['shipping_fullname'],
                'shipping_telephone'      => $order_query->row['shipping_telephone'],
                                'shipping_company'        => $order_query->row['shipping_company'],
                                'shipping_address_1'      => $order_query->row['shipping_address_1'],
                                'shipping_address_2'      => $order_query->row['shipping_address_2'],
                                'shipping_postcode'       => $order_query->row['shipping_postcode'],
                'shipping_city_id'        => $order_query->row['shipping_city_id'],
                                'shipping_city'           => $order_query->row['shipping_city'],
                                'shipping_zone_id'        => $order_query->row['shipping_zone_id'],
                                'shipping_zone'           => $order_query->row['shipping_zone'],
                                'shipping_zone_code'      => $shipping_zone_code,
                                'shipping_country_id'     => $order_query->row['shipping_country_id'],
                                'shipping_country'        => $order_query->row['shipping_country'],
                'shipping_county_id'      => $order_query->row['shipping_county_id'],
                'shipping_county'         => $order_query->row['shipping_county'],
                                'shipping_iso_code_2'     => $shipping_iso_code_2,
                                'shipping_iso_code_3'     => $shipping_iso_code_3,
                                'shipping_address_format' => $order_query->row['shipping_address_format'],
                                'shipping_custom_field'   => json_decode($order_query->row['shipping_custom_field'], true),
                                'shipping_method'         => $order_query->row['shipping_method'],
                                'shipping_code'           => $order_query->row['shipping_code'],
                                'comment'                 => $order_query->row['comment'],
                                'total'                   => $order_query->row['total'],
                                'reward'                  => $reward,
                                'order_status_id'         => $order_query->row['order_status_id'],
                                'order_status'            => $order_query->row['order_status'],
                                'affiliate_id'            => $order_query->row['affiliate_id'],
                                'affiliate_fullname'     => $affiliate_fullname,
                                'commission'              => $order_query->row['commission'],
                                'language_id'             => $order_query->row['language_id'],
                                'language_code'           => $language_code,
                                'currency_id'             => $order_query->row['currency_id'],
                                'currency_code'           => $order_query->row['currency_code'],
                                'currency_value'          => $order_query->row['currency_value'],
                                'ip'                      => $order_query->row['ip'],
                                'forwarded_ip'            => $order_query->row['forwarded_ip'],
                                'user_agent'              => $order_query->row['user_agent'],
                                'accept_language'         => $order_query->row['accept_language'],
                                'date_added'              => $order_query->row['date_added'],
                                'date_modified'           => $order_query->row['date_modified']
                        );
                } else {
                        return;
                }
        }

        public function getOrders($data = array()) {
                $sql = "SELECT o.order_id, o.fullname AS customer, (SELECT os.name FROM " . DB_PREFIX . "order_status os WHERE os.order_status_id = o.order_status_id AND os.language_id = '" . (int)$this->config->get('config_language_id') . "') AS order_status, o.shipping_code, o.total, o.currency_code, o.currency_value, o.date_added, o.date_modified FROM `" . DB_PREFIX . "order` o";

                if (!empty($data['filter_order_status'])) {
                        $implode = array();

                        $order_statuses = explode(',', $data['filter_order_status']);

                        foreach ($order_statuses as $order_status_id) {
                                $implode[] = "o.order_status_id = '" . (int)$order_status_id . "'";
                        }

                        if ($implode) {
                                $sql .= " WHERE (" . implode(" OR ", $implode) . ")";
                        }
                } elseif (isset($data['filter_order_status_id']) && $data['filter_order_status_id'] !== '') {
                        $sql .= " WHERE o.order_status_id = '" . (int)$data['filter_order_status_id'] . "'";
                } else {
                        $sql .= " WHERE o.order_status_id > '0'";
                }

                if (!empty($data['filter_order_id'])) {
                        $sql .= " AND o.order_id = '" . (int)$data['filter_order_id'] . "'";
                }

                if (!empty($data['filter_customer'])) {
                        $sql .= " AND o.fullname LIKE '%" . $this->db->escape((string)$data['filter_customer']) . "%'";
                }

                if (!empty($data['filter_date_added'])) {
                        $sql .= " AND DATE(o.date_added) >= DATE('" . $this->db->escape((string)$data['filter_date_added']) . "')";
                }

                if (!empty($data['filter_date_modified'])) {
                        $sql .= " AND DATE(o.date_added) <= DATE('" . $this->db->escape((string)$data['filter_date_modified']) . "')";
                }

        if (!empty($data['filter_total'])) {
            if (stripos($data['filter_total'], '-')) {
                $totals = explode('-', $data['filter_total']);
                $from_total = $totals[0];
                $to_total = $totals[1];
                $sql .= " AND o.total >= '" . (float)$from_total . "' AND o.total <= '" . (float)$to_total . "'";
            } else {
                $sql .= " AND o.total = '" . (float)$data['filter_total'] . "'";
            }
        }

                $sort_data = array(
                        'o.order_id',
                        'customer',
                        'order_status',
                        'o.date_added',
                        'o.date_modified',
                        'o.total'
                );

                if (isset($data['sort']) && in_array($data['sort'], $sort_data)) {
                        $sql .= " ORDER BY " . $data['sort'];
                } else {
                        $sql .= " ORDER BY o.order_id";
                }

                if (isset($data['order']) && ($data['order'] == 'DESC')) {
                        $sql .= " DESC";
                } else {
                        $sql .= " ASC";
                }

                if (isset($data['start']) || isset($data['limit'])) {
                        if ($data['start'] < 0) {
                                $data['start'] = 0;
                        }

                        if ($data['limit'] < 1) {
                                $data['limit'] = 20;
                        }

                        $sql .= " LIMIT " . (int)$data['start'] . "," . (int)$data['limit'];
                }

                $query = $this->db->query($sql);

                return $query->rows;
        }

        public function getOrderProducts($order_id) {
                $query = $this->db->query("SELECT * FROM " . DB_PREFIX . "order_product WHERE order_id = '" . (int)$order_id . "'");

                return $query->rows;
        }

        public function getOrderOptions($order_id, $order_product_id) {
                $query = $this->db->query("SELECT * FROM " . DB_PREFIX . "order_option WHERE order_id = '" . (int)$order_id . "' AND order_product_id = '" . (int)$order_product_id . "'");

                return $query->rows;
        }

        public function getOrderVouchers($order_id) {
                $query = $this->db->query("SELECT * FROM " . DB_PREFIX . "order_voucher WHERE order_id = '" . (int)$order_id . "'");

                return $query->rows;
        }

        public function getOrderVoucherByVoucherId($voucher_id) {
                $query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "order_voucher` WHERE voucher_id = '" . (int)$voucher_id . "'");

                return $query->row;
        }

        public function getOrderTotals($order_id) {
                $query = $this->db->query("SELECT * FROM " . DB_PREFIX . "order_total WHERE order_id = '" . (int)$order_id . "' ORDER BY sort_order");

                return $query->rows;
        }

        public function getTotalOrders($data = array()) {
                $sql = "SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order`";

                if (!empty($data['filter_order_status'])) {
                        $implode = array();

                        $order_statuses = explode(',', $data['filter_order_status']);

                        foreach ($order_statuses as $order_status_id) {
                                $implode[] = "order_status_id = '" . (int)$order_status_id . "'";
                        }

                        if ($implode) {
                                $sql .= " WHERE (" . implode(" OR ", $implode) . ")";
                        }
                } elseif (isset($data['filter_order_status_id']) && $data['filter_order_status_id'] !== '') {
                        $sql .= " WHERE order_status_id = '" . (int)$data['filter_order_status_id'] . "'";
                } else {
                        $sql .= " WHERE order_status_id > '0'";
                }

                if (!empty($data['filter_order_id'])) {
                        $sql .= " AND order_id = '" . (int)$data['filter_order_id'] . "'";
                }

                if (!empty($data['filter_customer'])) {
                        $sql .= " AND fullname LIKE '%" . $this->db->escape((string)$data['filter_customer']) . "%'";
                }

                if (!empty($data['filter_date_added'])) {
                        $sql .= " AND DATE(date_added) >= DATE('" . $this->db->escape((string)$data['filter_date_added']) . "')";
                }

                if (!empty($data['filter_date_modified'])) {
                        $sql .= " AND DATE(date_modified) <= DATE('" . $this->db->escape((string)$data['filter_date_modified']) . "')";
                }

        if (!empty($data['filter_total'])) {
            if (stripos($data['filter_total'], '-')) {
                $totals = explode('-', $data['filter_total']);
                $from_total = $totals[0];
                $to_total = $totals[1];
                $sql .= " AND total >= '" . (float)$from_total . "' AND total <= '" . (float)$to_total . "'";
            } else {
                $sql .= " AND total = '" . (float)$data['filter_total'] . "'";
            }
        }

                $query = $this->db->query($sql);

                return $query->row['total'];
        }

        public function getTotalOrdersByStoreId($store_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE store_id = '" . (int)$store_id . "'");

                return $query->row['total'];
        }

        public function getTotalOrdersByOrderStatusId($order_status_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE order_status_id = '" . (int)$order_status_id . "' AND order_status_id > '0'");

                return $query->row['total'];
        }

        public function getTotalOrdersByProcessingStatus() {
                $implode = array();

                $order_statuses = $this->config->get('config_processing_status');

                foreach ($order_statuses as $order_status_id) {
                        $implode[] = "order_status_id = '" . (int)$order_status_id . "'";
                }

                if ($implode) {
                        $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE " . implode(" OR ", $implode));

                        return $query->row['total'];
                } else {
                        return 0;
                }
        }

        public function getTotalOrdersByCompleteStatus() {
                $implode = array();

                $order_statuses = $this->config->get('config_complete_status');

                foreach ($order_statuses as $order_status_id) {
                        $implode[] = "order_status_id = '" . (int)$order_status_id . "'";
                }

                if ($implode) {
                        $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE " . implode(" OR ", $implode) . "");

                        return $query->row['total'];
                } else {
                        return 0;
                }
        }

        public function getTotalOrdersByLanguageId($language_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE language_id = '" . (int)$language_id . "' AND order_status_id > '0'");

                return $query->row['total'];
        }

        public function getTotalOrdersByCurrencyId($currency_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM `" . DB_PREFIX . "order` WHERE currency_id = '" . (int)$currency_id . "' AND order_status_id > '0'");

                return $query->row['total'];
        }

        public function getTotalSales($data = array()) {
                $sql = "SELECT SUM(total) AS total FROM `" . DB_PREFIX . "order`";

                if (!empty($data['filter_order_status'])) {
                        $implode = array();

                        $order_statuses = explode(',', $data['filter_order_status']);

                        foreach ($order_statuses as $order_status_id) {
                                $implode[] = "order_status_id = '" . (int)$order_status_id . "'";
                        }

                        if ($implode) {
                                $sql .= " WHERE (" . implode(" OR ", $implode) . ")";
                        }
                } elseif (isset($data['filter_order_status_id']) && $data['filter_order_status_id'] !== '') {
                        $sql .= " WHERE order_status_id = '" . (int)$data['filter_order_status_id'] . "'";
                } else {
                        $sql .= " WHERE order_status_id > '0'";
                }

                if (!empty($data['filter_order_id'])) {
                        $sql .= " AND order_id = '" . (int)$data['filter_order_id'] . "'";
                }

                if (!empty($data['filter_customer'])) {
                        $sql .= " AND fullname LIKE '%" . $this->db->escape((string)$data['filter_customer']) . "%'";
                }

                if (!empty($data['filter_date_added'])) {
                        $sql .= " AND DATE(date_added) = DATE('" . $this->db->escape((string)$data['filter_date_added']) . "')";
                }

                if (!empty($data['filter_date_modified'])) {
                        $sql .= " AND DATE(date_modified) = DATE('" . $this->db->escape((string)$data['filter_date_modified']) . "')";
                }

                if (!empty($data['filter_total'])) {
                        $sql .= " AND total = '" . (float)$data['filter_total'] . "'";
                }

                $query = $this->db->query($sql);

                return $query->row['total'];
        }

        public function createInvoiceNo($order_id, $invoice_no) {

                        $this->db->query("UPDATE `" . DB_PREFIX . "order` SET invoice_no = '" . (string)$invoice_no . "', invoice_prefix = '" . $this->db->escape($order_info['invoice_prefix']) . "' WHERE order_id = '" . (int)$order_id . "'");

                        return $order_info['invoice_prefix'] . $invoice_no;
        }
        public function getOrderHistories($order_id, $start = 0, $limit = 10) {
                if ($start < 0) {
                        $start = 0;
                }

                if ($limit < 1) {
                        $limit = 10;
                }

                $query = $this->db->query("SELECT oh.date_added, os.name AS status, oh.comment, oh.notify FROM " . DB_PREFIX . "order_history oh LEFT JOIN " . DB_PREFIX . "order_status os ON oh.order_status_id = os.order_status_id WHERE oh.order_id = '" . (int)$order_id . "' AND os.language_id = '" . (int)$this->config->get('config_language_id') . "' ORDER BY oh.date_added DESC LIMIT " . (int)$start . "," . (int)$limit);

                return $query->rows;
        }

        public function getTotalOrderHistories($order_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM " . DB_PREFIX . "order_history WHERE order_id = '" . (int)$order_id . "'");

                return $query->row['total'];
        }

        public function getTotalOrderHistoriesByOrderStatusId($order_status_id) {
                $query = $this->db->query("SELECT COUNT(*) AS total FROM " . DB_PREFIX . "order_history WHERE order_status_id = '" . (int)$order_status_id . "'");

                return $query->row['total'];
        }

        public function getEmailsByProductsOrdered($products, $start, $end) {
                $implode = array();

                foreach ($products as $product_id) {
                        $implode[] = "op.product_id = '" . (int)$product_id . "'";
                }

                $query = $this->db->query("SELECT DISTINCT email FROM `" . DB_PREFIX . "order` o LEFT JOIN " . DB_PREFIX . "order_product op ON (o.order_id = op.order_id) WHERE (" . implode(" OR ", $implode) . ") AND o.order_status_id <> '0' LIMIT " . (int)$start . "," . (int)$end);

                return $query->rows;
        }

        public function getTotalEmailsByProductsOrdered($products) {
                $implode = array();

                foreach ($products as $product_id) {
                        $implode[] = "op.product_id = '" . (int)$product_id . "'";
                }

                $query = $this->db->query("SELECT COUNT(DISTINCT email) AS total FROM `" . DB_PREFIX . "order` o LEFT JOIN " . DB_PREFIX . "order_product op ON (o.order_id = op.order_id) WHERE (" . implode(" OR ", $implode) . ") AND o.order_status_id <> '0'");

                return $query->row['total'];
        }
}
```

