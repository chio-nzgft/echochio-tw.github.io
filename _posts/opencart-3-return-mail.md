---
layout: post
title: opencart 3.X 退貨郵件通知
date: 2018-07-11
tags: opencart 3.X EmailOneReturn
---


此為 opencart 3.X 的退貨郵件通知 XML ...看得懂就拿去用

```
<modification>
	<name>EmailOneReturn by echochio</name>
	<version>1.0</version>
	<link>http://echochio.com</link>
	<author>echochio</author>
	<code>echochio_mailonreturn</code>
	<file path="catalog/model/account/return.php">

		<operation>
			<search><![CDATA[public function addReturn($data) {]]></search>
			<add position="before"><![CDATA[
				public function getReturnReason($return_reason_id) {
					$query = $this->db->query("SELECT * FROM " . DB_PREFIX . "return_reason WHERE return_reason_id = '" . (int)$return_reason_id . "' AND language_id = '" . (int)$this->config->get('config_language_id') . "'");

					return $query->row;
				}
				]]></add>
		</operation>


		<operation>
			<search><![CDATA[public function addReturn($data) {]]></search>
			<add position="after"><![CDATA[
                                /* MailOnReturn */
                                $this->load->model('localisation/return_reason');

                                $return_email_subject = "客戶提交了退貨申請 - " . $this->db->escape($data['fullname']);
                                $return_email_text = "客戶提交了新的產品退貨申請.\n\n";
                                $return_email_text .= "客戶 : " . $this->db->escape($data['fullname']). "\n";

                                if (!empty($data['email'])) {
                                        $return_email_text .= 'Email : ' . $this->db->escape($data['email']) . "\n";
                                }

                                if (!empty($data['telephone'])) {
                                        $return_email_text .= '電話 : ' . $this->db->escape($data['telephone']) . "\n\n";
                                }

                                $return_email_text .= '訂單編號 : '  . $data['order_id'] . "\n";
                                $return_email_text .= '訂單日 : ' . $data['date_ordered'] . "\n";
                                $return_email_text .= '購買產品 : ' . $this->db->escape($data['product']) . "\n";
                                $return_email_text .= '產品型號 : ' . $this->db->escape($data['model']) . "\n\n";

                                $return_reason_info = $this->getReturnReason($data['return_reason_id']);

                                if (!empty($return_reason_info['name'])) {
                                        $return_email_text .= '退貨原因 : ' . $this->db->escape($return_reason_info['name']) . "\n";
                                }

                                if (!empty($data['comment'])) {
                                        $return_email_text .= '其他附註 : ' . $this->db->escape($data['comment']) . "\n\n";
                                }

                                $return_email_text .= '請登入購物網後台查看';

                        $mail = new Mail($this->config->get('config_mail_engine'));
                        $mail->parameter = $this->config->get('config_mail_parameter');
                        $mail->smtp_hostname = $this->config->get('config_mail_smtp_hostname');
                        $mail->smtp_username = $this->config->get('config_mail_smtp_username');
                        $mail->smtp_password = html_entity_decode($this->config->get('config_mail_smtp_password'), ENT_QUOTES, 'UTF-8');
                        $mail->smtp_port = $this->config->get('config_mail_smtp_port');
                        $mail->smtp_timeout = $this->config->get('config_mail_smtp_timeout');


                                $mail->setTo($this->config->get('config_email'));
                                $mail->setFrom($this->config->get('config_email'));
                                $mail->setSender($this->config->get('config_name'));
                                $mail->setSubject(html_entity_decode($return_email_subject, ENT_QUOTES, 'UTF-8'));
                                $mail->setText(html_entity_decode($return_email_text, ENT_QUOTES, 'UTF-8'));
                                $mail->send();

                                // for user mail

                                $return_email_subject = $this->db->escape($data['fullname']) . " - 提交了退貨申請";
                                $return_email_text = "您提交了新的產品退貨申請.\n\n";
                                $return_email_text .= "姓名 : " . $this->db->escape($data['fullname']). "\n";

                                if (!empty($data['email'])) {
                                        $return_email_text .= 'Email : ' . $this->db->escape($data['email']) . "\n";
                                }

                                if (!empty($data['telephone'])) {
                                        $return_email_text .= '電話 : ' . $this->db->escape($data['telephone']) . "\n\n";
                                }

                                $return_email_text .= '訂單編號 : '  . $data['order_id'] . "\n";
                                $return_email_text .= '訂單日 : ' . $data['date_ordered'] . "\n";
                                $return_email_text .= '購買產品 : ' . $this->db->escape($data['product']) . "\n";
                                $return_email_text .= '產品型號 : ' . $this->db->escape($data['model']) . "\n\n";

                                $return_reason_info = $this->getReturnReason($data['return_reason_id']);

                                if (!empty($return_reason_info['name'])) {
                                        $return_email_text .= '退貨原因 : ' . $this->db->escape($return_reason_info['name']) . "\n";
                                }

                                if (!empty($data['comment'])) {
                                        $return_email_text .= '其他附註 : ' . $this->db->escape($data['comment']) . "\n\n";
                                }

                                $return_email_text .= '將會盡快處理您的訂單';

                        $mail = new Mail($this->config->get('config_mail_engine'));
                        $mail->parameter = $this->config->get('config_mail_parameter');
                        $mail->smtp_hostname = $this->config->get('config_mail_smtp_hostname');
                        $mail->smtp_username = $this->config->get('config_mail_smtp_username');
                        $mail->smtp_password = html_entity_decode($this->config->get('config_mail_smtp_password'), ENT_QUOTES, 'UTF-8');
                        $mail->smtp_port = $this->config->get('config_mail_smtp_port');
                        $mail->smtp_timeout = $this->config->get('config_mail_smtp_timeout');


                                $mail->setTo($this->db->escape($data['email']));
                                $mail->setFrom($this->config->get('config_email'));
                                $mail->setSender($this->config->get('config_name'));
                                $mail->setSubject(html_entity_decode($return_email_subject, ENT_QUOTES, 'UTF-8'));
                                $mail->setText(html_entity_decode($return_email_text, ENT_QUOTES, 'UTF-8'));
                                $mail->send();

                                /* End MailOnReturn */

			]]></add>
		</operation>
	</file>
</modification>
```
