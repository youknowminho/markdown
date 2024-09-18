\*\*Session 1: Permissions and Encryption Using Cross-Account SNS-SQS Integration\*\*

\---

\*\*Overview:\*\*

For Session 1, we'll focus on the \*\*Fan Club Membership Rewards\*\* branch of the Taylor Swift World Tour ticketing system. In this scenario, the \*\*FanClubAwardsQueue\*\* and the \*\*FanClubAwardProcessor\*\* Lambda function are managed by a separate department operating in a different AWS account. This setup mirrors real-world situations where organizations delegate certain functions (like loyalty programs) to specialized departments or external partners.

This session will provide hands-on experience with:

\- \*\*Cross-Account SNS to SQS Permissions\*\*: Configuring SNS in one AWS account to send messages to an SQS queue in another account.  
\- \*\*Encryption\*\*: Implementing server-side encryption with AWS Key Management Service (KMS) for both SNS topics and SQS queues, including cross-account key usage.

\---

\#\#\# \*\*Real-World Scenario:\*\*

In large organizations, it's common for different departments or business units to manage their own AWS accounts for billing, security, and administrative purposes. For example, the \*\*Membership Rewards Department\*\* might have its own AWS account to independently manage fan club benefits, membership data, and reward processing systems.

\---

\#\#\# \*\*Updated Architecture Components:\*\*

\*\*Account A (Primary Account):\*\*

\- \*\*SNS Topic\*\*: \*\*TourTicketingTopic\*\*  
  \- \*\*Role\*\*: Publishes messages related to ticket purchases.

\- \*\*Lambda Function\*\*: \*\*TicketPurchaseHandler\*\*  
  \- \*\*Role\*\*: Processes ticket purchases and publishes to \*\*TourTicketingTopic\*\*.

\*\*Account B (Membership Department Account):\*\*

\- \*\*SQS Queue\*\*: \*\*FanClubAwardsQueue\*\*  
  \- \*\*Role\*\*: Receives messages from \*\*TourTicketingTopic\*\* in Account A.  
  \- \*\*Encryption\*\*: Uses an AWS KMS key owned by Account B.

\- \*\*Lambda Function\*\*: \*\*FanClubAwardProcessor\*\*  
  \- \*\*Role\*\*: Processes messages from \*\*FanClubAwardsQueue\*\* to update membership rewards.

\---

\#\#\# \*\*Session Plan:\*\*

\*\*Objective:\*\*

By the end of this session, participants will:

1\. Understand how to set up cross-account permissions between SNS and SQS.  
2\. Configure encryption for SNS topics and SQS queues, including cross-account KMS key permissions.  
3\. Gain practical experience with IAM policies, resource policies, and key policies in a cross-account context.

\---

\#\#\# \*\*Step-by-Step Activities:\*\*

\#\#\#\# \*\*1. Set Up AWS Accounts:\*\*

\- \*\*Account A (Primary Account)\*\*: Represents the main ticketing system.  
\- \*\*Account B (Membership Account)\*\*: Represents the membership rewards department.

\*For the workshop, AWS Organizations or individual AWS accounts can be used to simulate the cross-account environment.\*

\---

\#\#\#\# \*\*2. Configure SNS Topic in Account A:\*\*

\- \*\*Create SNS Topic\*\*: \*\*TourTicketingTopic\*\*  
\- \*\*Enable Encryption\*\*:  
  \- Use an AWS KMS key (e.g., \*\*arn:aws:kms:region:AccountA:key/key-id\*\*).  
  \- Ensure the key policy allows SNS to use the key for encryption.

\---

\#\#\#\# \*\*3. Set Up SQS Queue in Account B:\*\*

\- \*\*Create SQS Queue\*\*: \*\*FanClubAwardsQueue\*\*  
\- \*\*Enable Encryption\*\*:  
  \- Use an AWS KMS key owned by Account B (e.g., \*\*arn:aws:kms:region:AccountB:key/key-id\*\*).  
  \- Configure the key policy to allow SQS to decrypt messages encrypted by Account A's SNS topic.

\---

\#\#\#\# \*\*4. Configure Cross-Account SNS to SQS Subscription:\*\*

\- \*\*In Account A\*\*:

  \- \*\*Set Up Subscription\*\*:  
    \- Create a subscription on \*\*TourTicketingTopic\*\* with the endpoint being the ARN of \*\*FanClubAwardsQueue\*\* in Account B.

\- \*\*In Account B\*\*:

  \- \*\*Modify SQS Queue Policy\*\*:  
    \- Update the \*\*FanClubAwardsQueue\*\* policy to allow SNS in Account A to send messages.  
    \- Example policy snippet:

      \`\`\`json  
      {  
        "Version": "2012-10-17",  
        "Id": "ExampleQueuePolicy",  
        "Statement": \[  
          {  
            "Sid": "Allow-SNS-SendMessage",  
            "Effect": "Allow",  
            "Principal": {  
              "AWS": "arn:aws:iam::AccountA:root"  
            },  
            "Action": "SQS:SendMessage",  
            "Resource": "arn:aws:sqs:region:AccountB:FanClubAwardsQueue",  
            "Condition": {  
              "ArnEquals": {  
                "aws:SourceArn": "arn:aws:sns:region:AccountA:TourTicketingTopic"  
              }  
            }  
          }  
        \]  
      }  
      \`\`\`

\---

\#\#\#\# \*\*5. Configure KMS Key Policies for Cross-Account Access:\*\*

\- \*\*In Account B (KMS Key for SQS Queue Encryption)\*\*:

  \- \*\*Modify Key Policy\*\* to allow decryption of messages encrypted with Account A's KMS key.

  \- Example key policy snippet:

    \`\`\`json  
    {  
      "Version": "2012-10-17",  
      "Id": "key-policy",  
      "Statement": \[  
        {  
          "Sid": "Allow Account B Use of the Key",  
          "Effect": "Allow",  
          "Principal": {  
            "AWS": "arn:aws:iam::AccountB:root"  
          },  
          "Action": \[  
            "kms:Decrypt",  
            "kms:DescribeKey"  
          \],  
          "Resource": "\*"  
        },  
        {  
          "Sid": "Allow Account A SNS to Encrypt",  
          "Effect": "Allow",  
          "Principal": {  
            "AWS": "arn:aws:iam::AccountA:root"  
          },  
          "Action": \[  
            "kms:Encrypt"  
          \],  
          "Resource": "\*"  
        }  
      \]  
    }  
    \`\`\`

\- \*\*In Account A (KMS Key for SNS Topic Encryption)\*\*:

  \- Ensure the key policy allows SNS to use the key for message encryption.

\---

\#\#\#\# \*\*6. Update Lambda Functions and IAM Roles:\*\*

\- \*\*TicketPurchaseHandler (Account A)\*\*:

  \- Ensure it has permissions to publish messages to \*\*TourTicketingTopic\*\*.

\- \*\*FanClubAwardProcessor (Account B)\*\*:

  \- Requires permissions to read messages from \*\*FanClubAwardsQueue\*\*.  
  \- Needs decrypt permissions for the KMS key used by \*\*FanClubAwardsQueue\*\*.

\---

\#\#\#\# \*\*7. Testing the Setup:\*\*

\- \*\*Simulate a Ticket Purchase\*\*:

  \- Invoke \*\*TicketPurchaseHandler\*\* with a payload indicating a fan club member purchase.

\- \*\*Verify Message Flow\*\*:

  \- Check that the message is published to \*\*TourTicketingTopic\*\* in Account A.  
  \- Confirm that the message arrives in \*\*FanClubAwardsQueue\*\* in Account B.

\- \*\*Decrypt and Process the Message\*\*:

  \- Ensure \*\*FanClubAwardProcessor\*\* successfully retrieves and processes the message.

\---

\#\#\# \*\*Educational Outcomes:\*\*

Participants will learn:

\- \*\*Cross-Account SNS to SQS Integration\*\*:

  \- How to configure SNS topics to send messages to SQS queues in a different AWS account.  
  \- Understanding of the necessary IAM roles, policies, and resource policies.

\- \*\*Encryption in Cross-Account Context\*\*:

  \- How to set up KMS keys for encryption and decryption across AWS accounts.  
  \- Managing key policies to allow cross-account access securely.

\- \*\*Real-World Application\*\*:

  \- Recognize how departments within an organization can securely share data while maintaining autonomy.  
  \- Apply best practices for inter-account communication and data protection.

\---

\#\#\# \*\*Additional Considerations:\*\*

\#\#\#\# \*\*Message Filtering (Optional Enhancement):\*\*

\- \*\*Purpose\*\*: Ensure that only relevant messages are delivered to \*\*FanClubAwardsQueue\*\*.

\- \*\*Implementation\*\*:

  \- Use message attributes and subscription filters on the SNS topic to route only fan club-related messages to \*\*FanClubAwardsQueue\*\*.

\---

\#\#\#\# \*\*Error Handling and Monitoring:\*\*

\- \*\*Dead-Letter Queues (DLQs)\*\*:

  \- Set up DLQs for \*\*FanClubAwardsQueue\*\* to capture messages that fail processing.

\- \*\*Logging and Metrics\*\*:

  \- Use CloudWatch to monitor message delivery status, encryption/decryption metrics, and Lambda function performance.

\---

\#\#\# \*\*Summary of Key Learnings:\*\*

\- \*\*IAM Policies and Resource Policies\*\*:

  \- Understand the difference between identity-based policies and resource-based policies.  
  \- Learn how to apply these policies for cross-account access.

\- \*\*KMS Key Policies\*\*:

  \- Grasp how key policies control access to encryption keys.  
  \- Configure key policies to allow services and principals in other accounts to use the keys.

\- \*\*Security Best Practices\*\*:

  \- Implement the principle of least privilege.  
  \- Ensure data is encrypted in transit and at rest, even across accounts.

\- \*\*Troubleshooting Cross-Account Issues\*\*:

  \- Identify and resolve common pitfalls, such as missing permissions or incorrect key policies.

\---

\#\#\# \*\*Real-World Relevance:\*\*

This exercise mirrors situations where companies:

\- \*\*Partner with External Organizations\*\*:

  \- For example, collaborating with a third-party marketing firm that manages customer loyalty programs.

\- \*\*Maintain Separate AWS Accounts for Security and Billing\*\*:

  \- Departments may have separate accounts to isolate resources and manage costs independently.

\- \*\*Need Secure Cross-Account Communication\*\*:

  \- Ensuring data is securely transmitted and only accessible to authorized parties.

\---

\#\#\# \*\*Conclusion:\*\*

By setting up a cross-account, encrypted communication channel between the SNS topic in Account A and the SQS queue in Account B, participants will gain practical experience in configuring and securing AWS services in a multi-account environment. This hands-on exercise reinforces critical concepts in IAM, resource policies, KMS key management, and the intricacies of cross-account permissions, all within the engaging context of the Taylor Swift World Tour ticketing system.

\---

\#\#\# \*\*Next Steps for Participants:\*\*

1\. \*\*Review IAM and KMS Documentation\*\*:

   \- Familiarize yourself with IAM policies, resource policies, and KMS key policies.

2\. \*\*Hands-On Implementation\*\*:

   \- Follow the step-by-step guide to set up the cross-account integration.

3\. \*\*Experiment with Variations\*\*:

   \- Try changing policies to see how permissions affect the system.  
   \- Implement additional security measures, like VPC endpoints for private communication.

4\. \*\*Prepare for Session 2\*\*:

   \- Monitor the system using CloudWatch.  
   \- Collect metrics that will be analyzed in the next session.

\---

\*\*By the end of this session, you'll have a solid understanding of how to securely connect AWS services across accounts, a vital skill for any AWS professional dealing with complex, multi-account architectures.\*\*