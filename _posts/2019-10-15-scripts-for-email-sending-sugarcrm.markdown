---
layout: post
title:  "Send Email - SugarCRM"
date:   2019-10-08 12:16:29 +0500
categories: sugarcrm
---
Please see the scripts here:
#### SendNotificationHelper.php

```php
<?php

require_once('custom/include/utils/SendEmail.php');

/**
 * SendNotificationHelper class for all helper methods of sending custom notifications
 *
 * Example: AUDIHelper::getSendNotificationHelper()->sendEmail($args)
 */
class SendNotificationHelper
{
    /**
     *
     * @param type $data
     * @return boolean
     */
    public function sendEmail($data)
    {
        $admin = Administration::getSettings();
        try {
            $email = new Email();
            $email->email2init();
            $_REQUEST['action'] = "EmailUIAjax";
            $_REQUEST['emailUIAction'] = "sendEmail";
            $_REQUEST['module'] = "Emails";
            $_REQUEST['saveToSugar'] = "1";
            $_REQUEST['sendCharset'] = "ISO-8859-1";
            $_REQUEST['to_pdf'] = "true";

            if (!empty($data['record_id']) && !empty($data['record_module_name'])) {
                $_REQUEST['parent_id'] = $data['record_id'];
                $_REQUEST['parent_type'] = $data['record_module_name'];
            }

            $_REQUEST['sendSubject'] = $data['email_template_subject'];
            $_REQUEST['sendTo'] = $data['email_address'];
            $_REQUEST['sendDescription'] = $data['email_template_body'];
            $_REQUEST['sendCc'] = '';
            $_REQUEST['sendBcc'] = '';
            $_REQUEST['composeType'] = '';
            $_REQUEST['setEditor'] = '1';
            $_REQUEST['fromAccount'] = '';

            $response = $email->email2Send($_REQUEST);

            if ($response) {
                if (!empty($data['populate_bean_after_email_send'])) {
                    $this->populateBeanAfterEmailSend($data['populate_bean_after_email_send']);
                }
                $GLOBALS['log']->fatal("Alert Notification Email Sent");
            } else {
                $GLOBALS['log']->fatal("Unable to Alert Notificaiton Email");
            }
            
            return $response;
        } catch (Exception $e) {
            $GLOBALS['log']->fatal($e->getMessage());
        }
    }

    /**
     * This function is used to update the bean data after successful email send
     *
     * @param array $populateData
     */
    protected function populateBeanAfterEmailSend($populateData)
    {
        if (empty($populateData)) {
            return;
        }
        foreach ($populateData as $record_id => $populateDataProperties) {
            if (!empty($record_id)) {
                $bean = null;
                foreach ($populateDataProperties as $key => $property) {
                    if ((is_null($bean) || empty($bean->id)) && !empty($property['module_name'])) {
                        $bean = $this->getModuleBean(array('module' => $property['module_name'], 'id' => $record_id));
                    }
                    if (!empty($property['field_name'])) {
                        $bean->$property['field_name'] = $property['field_value'];
                    }
                }
                if (!empty($bean->id)) {
                    $bean->save();
                }
            }
        }
    }

    /**
     *
     * @param type $templateId
     * @param type $email_address
     * @param type $bean
     * @param type $templateObj
     */
    protected function createJobQueue($templateId, $email_address, $bean, $templateObj)
    {
        scheduleEmailUtil($templateId, $email_address, $bean, $templateObj);
    }
}
```

#### SendEmail.php
```php
<?php

/**
 * This function is used to send emails using email templates and schedular job
 *
 * @param string $templateId : ID of the email template
 * @param array $emailAddress : email address, to whom email will be sent
 * @param SugarBean $bean : it will be used to parse email template
 *
 */

function scheduleEmailUtil($templateId, $emailAddress, $bean, $templateObj = null, $priority = AudiJobQueue::JOB_PRIORITY_MEDIUM)
{
    try {
        if (!empty($emailAddress) && (!empty($templateId) || !empty($templateObj))) {
            $htmlBody = '';
            if(empty($templateObj))
            {
                $templateObj = getEmailTemplateObject($templateId);
                $htmlBody = $templateObj->body_html;
                $htmlBody = parse_alert_template($bean, $htmlBody);
            }
            else
            {
                $htmlBody = $templateObj->body_html;
            }
            $job = new SchedulersJob();
            $job->name = "Send Email Job";
            // As it accepts strings so encode and serialize the array
            $module_name = '';
            $record_id = '';
            if (!empty($bean)) {
                $module_name = $bean->module_name;
                $record_id = $bean->id;
                if (!empty($bean->populate_bean_after_email_send)) {
                    $data['populate_bean_after_email_send'] = $bean->populate_bean_after_email_send;
                }
            }
            $data = array(
                'email_address' => $emailAddress,
                'email_template_body' => $htmlBody,
                'email_template_body_text' => $templateObj->body,
                'email_template_subject' => $templateObj->subject,
                'record_module_name' => $module_name,
                'record_id' => $record_id,
            );
            
            $job->data = base64_encode(serialize($data));
            $job->target = "function::sendEmailJob";
            $user = BeanFactory::getBean('Users');
            $job->assigned_user_id = $user->getSystemUser()->id;

            $jq = new AudiJobQueue();
            $jobid = $jq->submitJob($job, $priority);
        } else {
            throw new Exception("Email address or eamil template cannot be empty. source: scheduleEmailUtil().");
        }
    } catch (Exception $ex) {
        $GLOBALS['log']->fatal($ex->getMessage());
    }
}
function getEmailTemplateObject($templateId, $type = 'workflow')
{
    $templateObj = new EmailTemplate();
    $templateObj->retrieve_by_string_fields(array('id' => $templateId, 'type' => $type));
    return $templateObj;
}
function getCurrentLanguage($user = null)
{
    if (!empty($user) && !empty($user->preferred_language)) {        
        return $user->preferred_language;
    }

    if (!empty($GLOBALS['sugar_config']['default_language'])) {
        return $GLOBALS['sugar_config']['default_language'];
    }

    return 'de_DE';
}
function getUserPhrase($salutation = '', $user = null)
{
    if (empty($salutation)) {
        return $salutation;
    }
    $currentLanguage = getCurrentLanguage($user);
    $app_list_strings = return_app_list_strings_language($currentLanguage);
    if (!empty($app_list_strings) && isset($app_list_strings['phrase_salutation_mapping_dom'][$salutation])) {
        return $app_list_strings['phrase_salutation_mapping_dom'][$salutation];
    }
    return '';
}
function getTranslatedValue($key = '', $dom = '', $user = null)
{
    if (empty($key) || empty($dom)){
        return $key;
    }
    if (empty($user)) {
        global $current_user;
        $user = $current_user;
    }
    $currentLanguage = getCurrentLanguage($user);
    $app_list_strings = return_app_list_strings_language($currentLanguage);
    if (!empty($app_list_strings) && isset($app_list_strings[$dom][$key])) {
        return $app_list_strings[$dom][$key];
    }
    return '';
}
```
