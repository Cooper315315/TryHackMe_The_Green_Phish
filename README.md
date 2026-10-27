# The Greenholt Phish — Email Investigation Report

> A SOC investigation write-up for the TryHackMe room “The Greenholt Phish”.
> This report contains room spoilers. No live malware is included.

## 1. Executive Summary

A sales executive at Greenholt PLC reported a suspicious email apparently sent by a known customer. The message contained a generic greeting, an unexpected request for a money transfer, and an unsolicited attachment. These characteristics did not match the customer’s usual communication style.

Investigation findings identified a difference between the sender’s email address and the reply destination. The attachment also exhibited a file-type discrepancy: its filename contained `PDF` and ended in `.CAB`, while its reported actual format was RAR.

**Assessment:** The available findings support treating the message as a suspected phishing attempt involving a financial-transfer pretext and a suspicious attachment.

The supplied evidence does not establish a malware family, successful payload execution, credential theft, or endpoint compromise.

## 2. Scope and Evidence Basis

This report documents the provided incident scenario and the twelve completed room answers.

The original email, full headers, attachment bytes, screenshots, and VirusTotal report were not available for independent examination during report preparation.

Consequently:

- Artifact values are recorded as supplied lab findings rather than independently verified observations.
- Commands are included as reproduction guidance, not as evidence of execution during report preparation.
- IP ownership, DNS records, and VirusTotal metadata reflect the supplied room answers and may differ from current external results.
- No attachment was executed, extracted, or uploaded during report preparation.
- Interpretations and recommended follow-up actions are distinguished from confirmed findings.

## 3. Incident Background

| Field | Details |
| --- | --- |
| Organization | Greenholt PLC |
| Reporting employee | Sales executive |
| Claimed sender relationship | Known customer |
| Initial warning signs | Generic greeting, unexpected money-transfer request, unsolicited attachment |
| Escalation destination | Security Operations Center |
| Investigation objectives | Extract email artifacts, investigate source and authenticity, assess the attachment |

The initial concern was not simply that the email requested a transfer. The request was unexpected and appeared alongside other departures from the customer’s normal communication style.

These observations justified escalation for further investigation but did not independently prove maliciousness.

## 4. Key Artifacts and Findings

| Question | Artifact | Supplied finding |
| --- | --- | --- |
| Q1 | Transfer reference number in the subject | `09674321` |
| Q2 | Sender display name | `Mr. James Jackson` |
| Q3 | Sender email address | `info@mutawamarine.com` |
| Q4 | Reply destination | `info.mutawamarine@mail.com` |
| Q5 | Originating IP address | `192.119.71.157` |
| Q6 | Reported owner of the originating IP | `HostPapa` |
| Q7 | SPF record for the queried Return-Path domain | `v=spf1 include:spf.protection.outlook.com -all` |
| Q8 | DMARC record returned by the room lookup | `v=DMARC1; p=quarantine; fo=1` |
| Q9 | Attachment filename | `SWT_#09674321____PDF__.CAB` |
| Q10 | Attachment SHA256 | `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f` |
| Q11 | VirusTotal-reported attachment size | `400.26 KB` |
| Q12 | Actual attachment type | `RAR` |

The supplied findings do not include the complete subject line, exact Return-Path address or domain, message timestamp, Message-ID, full Received header chain, or receiving-system authentication results.

## 5. Email Content and Header Analysis

### 5.1 Financial-Transfer Pretext

The email requested an unexpected money transfer and included the reference number:

    09674321

The attachment filename contained the same reference number:

    SWT_#09674321____PDF__.CAB

This connection made the attachment appear relevant to the transfer request.

A matching reference number is not proof of legitimacy. In this scenario, it contributes to a coherent payment-related pretext that could encourage the recipient to trust or open the attachment.

### 5.2 Sender Display Name and Address

The supplied sender identity was:

    Mr. James Jackson <info@mutawamarine.com>

The display name presents a recognizable personal identity, while the email address appears to use a business domain.

Neither field, on its own, establishes who controlled or sent the message. Their authenticity must be assessed using additional evidence, including trusted message headers, authentication results, and independent confirmation with the customer.

### 5.3 Reply Destination Mismatch

The email used different sender and reply addresses:

| Field | Value |
| --- | --- |
| Sender address | `info@mutawamarine.com` |
| Reply destination | `info.mutawamarine@mail.com` |
| Sender domain | `mutawamarine.com` |
| Reply destination domain | `mail.com` |

The reply address repeats `mutawamarine` in the local part, before the `@` symbol. However, its actual domain is `mail.com`, not `mutawamarine.com`.

This distinction matters because a recipient could recognize the business-related text without noticing that replies would be sent to a separate domain.

The mismatch is consistent with possible reply redirection. In the context of an unexpected financial request, it strengthens suspicion that subsequent correspondence could be diverted away from the apparent customer.

A different reply destination is not automatically malicious; legitimate organizations sometimes use alternate reply addresses. It must be assessed alongside the other evidence.

### 5.4 Originating IP Address

The supplied originating IP address was:

    192.119.71.157

The reported owner was:

    HostPapa

This finding provides an infrastructure pivot for further investigation. It does not identify the person responsible for sending the email.

The original header chain was not available for independent review. Therefore, this report cannot confirm how the originating IP was selected or distinguish trusted receiving-server observations from potentially untrusted header content.

## 6. Email Authentication Analysis

### 6.1 SPF Record

The supplied SPF record was:

    v=spf1 include:spf.protection.outlook.com -all

| Component | Interpretation |
| --- | --- |
| `v=spf1` | Identifies the record as an SPF version 1 policy |
| `include:spf.protection.outlook.com` | Incorporates the referenced SPF policy into the authorization evaluation |
| `-all` | Specifies a hard-fail result for sources that do not match an authorized mechanism |

This record describes a sending-authorization policy. It does not establish the SPF result for the investigated message.

An SPF conclusion would require the relevant sender identity, connecting IP, applicable DNS information, and evaluation context. The supplied answers do not include the exact Return-Path domain or the receiving system’s SPF result.

Accordingly, this report does not claim that the email passed or failed SPF.

The reported IP owner should not be compared with the policy text as a substitute for an actual SPF evaluation.

### 6.2 DMARC Record

The supplied DMARC record was:

    v=DMARC1; p=quarantine; fo=1

| Component | Interpretation |
| --- | --- |
| `v=DMARC1` | Identifies the record as a DMARC policy |
| `p=quarantine` | Requests quarantine treatment for messages that fail DMARC |
| `fo=1` | Specifies an authentication-failure reporting option |

The presence of this policy does not establish that the investigated message failed DMARC or that the receiving system quarantined it.

The room question describes a DMARC lookup using the Return-Path domain. However, actual DMARC evaluation concerns alignment with the visible From domain. A message passes DMARC when SPF or DKIM passes with the required domain alignment.

The exact Return-Path domain and message-specific authentication results were not supplied. Therefore, the applicability of the queried record and the email’s DMARC outcome cannot be confirmed from the answers alone.

The `fo=1` setting is not evidence that an authentication-failure report was generated or received.

## 7. Attachment Analysis

### 7.1 Attachment Filename

The supplied filename was:

    SWT_#09674321____PDF__.CAB

Three characteristics are relevant:

1. It contains the same transfer reference as the subject.
2. It includes the text `PDF`, which may make the attachment appear document-related.
3. Its final extension is `.CAB`, not `.pdf`.

The reported actual format was:

    RAR

The attachment therefore exhibited a discrepancy between its apparent presentation, filename extension, and reported underlying format.

| Attribute | Observation |
| --- | --- |
| Document-like text | `PDF` |
| Final filename extension | `.CAB` |
| Reported actual file type | `RAR` |

This discrepancy is consistent with file-type disguise. It supports suspicion but does not establish the contents of the archive or prove that a contained file is malicious.

### 7.2 SHA256 Hash

The supplied SHA256 hash was:

    2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f

This value provides an exact-match pivot for comparing the attachment with other copies and searching for an existing threat-intelligence report.

To reproduce the hash calculation after saving the attachment in the isolated lab environment:

    sha256sum -- 'SWT_#09674321____PDF__.CAB'

The resulting hash should be compared character-for-character with the recorded value.

### 7.3 File-Type Verification

To inspect the file’s detected format in the lab environment:

    file -- 'SWT_#09674321____PDF__.CAB'

The command’s output should be recorded as evidence rather than relying on the filename extension.

These commands are reproduction guidance. Their output was not collected during preparation of this report.

### 7.4 VirusTotal Findings

The supplied VirusTotal-related findings were:

| Property | Supplied value |
| --- | --- |
| File size | `400.26 KB` |
| Actual file type | `RAR` |

No detection ratio, engine verdicts, behavioral analysis, malware-family identification, or archive contents were supplied.

Consequently, this report records the attachment as suspicious rather than claiming it is confirmed malware.

## 8. Evidence Assessment

| Evidence | Investigative significance | Limitation |
| --- | --- | --- |
| Generic greeting | Inconsistent with the reported customer’s normal communication style | Full message text unavailable |
| Unexpected transfer request | Supports a financial social-engineering hypothesis | Does not independently prove fraud |
| Different sender and reply domains | Consistent with possible reply redirection | Alternate reply routing can be legitimate |
| Shared reference in subject and attachment | Connects the attachment to the transfer pretext | Does not establish authenticity |
| PDF-like text in the filename | May encourage the recipient to perceive the attachment as a document | Recipient perception was not observed |
| CAB extension with reported RAR format | Consistent with file-type disguise | Archive contents were not supplied |
| Originating IP and reported owner | Provides an infrastructure investigation pivot | Does not establish threat-actor identity |
| SPF and DMARC policy records | Provides authentication-policy context | Does not establish message-specific authentication outcomes |

Taken together, the findings support a suspected phishing assessment more strongly than any single indicator would in isolation.

## 9. Investigation Pivots

The following artifacts can support searches across email telemetry, endpoint evidence, and existing threat-intelligence records.

Their inclusion does not mean that every associated domain, IP address, or service is malicious.

| Artifact type | Value | Suggested investigative use |
| --- | --- | --- |
| Sender address | `info@mutawamarine.com` | Search for related messages and compare sender patterns |
| Reply address | `info.mutawamarine@mail.com` | Identify messages directing responses to the same destination |
| Originating IP | `192.119.71.157` | Correlate relevant mail events and investigate infrastructure context |
| Transfer reference | `09674321` | Search subject lines, message bodies, and attachment names |
| Filename | `SWT_#09674321____PDF__.CAB` | Locate matching attachment names |
| SHA256 | `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f` | Locate exact matching files and existing analysis records |

IP-based findings should be interpreted cautiously, particularly where shared hosting infrastructure may be involved.

## 10. Recommended SOC Follow-up

The following are proposed actions, not actions completed during this write-up.

### 10.1 Preserve Evidence

- Preserve the original email in its native format.
- Retain complete message headers and the attachment.
- Record collection timestamps, artifact hashes, and evidence-handling details.
- Preserve relevant mail-security and endpoint telemetry.

### 10.2 Contain the Message

- Quarantine the reported message according to organizational procedures.
- Avoid replying to the suspicious email.
- Do not open or execute the attachment on a production workstation.
- Search for related messages before determining the scope of any wider removal action.

### 10.3 Validate the Customer Request

- Contact the customer using an independently established phone number or trusted communication channel.
- Confirm whether the customer sent the message.
- Verify whether the transfer request and attachment were expected.
- Do not rely on contact details supplied within the suspicious email.

### 10.4 Validate Source and Authentication

- Review the full Received header chain.
- Identify which header observations were added by trusted receiving infrastructure.
- Confirm the exact Return-Path and visible From domains.
- Review SPF, DKIM, and DMARC results recorded by the receiving system.
- Distinguish published DNS policies from actual message authentication outcomes.

### 10.5 Determine User Interaction and Impact

Establish whether the recipient:

- Replied to the message.
- Opened the attachment.
- Extracted the archive.
- Executed any contained file.
- Disclosed sensitive information.
- Initiated or completed a financial transfer.

If interaction occurred, escalate for endpoint investigation and coordinate with finance where payment activity is involved.

### 10.6 Continue Attachment Analysis Safely

- Search the SHA256 for existing analysis before considering a new submission.
- Follow organizational approval and data-handling requirements for external services.
- Perform additional analysis only in an approved isolated environment.
- Document archive contents, extracted-file hashes, and any observed behavior if further analysis is authorized.

## 11. Limitations and Unanswered Questions

The available material does not establish:

- The complete subject line or message body.
- The exact Return-Path address or domain.
- The complete and trusted message-routing path.
- The email’s actual SPF, DKIM, or DMARC results.
- Whether the apparent customer’s account was compromised.
- Who controlled the reply address.
- The archive’s contents.
- A VirusTotal detection ratio or confirmed malware family.
- Whether the recipient interacted with the attachment.
- Whether financial loss, data exposure, or endpoint compromise occurred.

These gaps should remain explicit rather than being filled with assumptions.

## 12. Conclusion

The supplied findings support classifying the message as suspected phishing.

The strongest combined indicators are the unexpected financial request, the separate reply destination, and the attachment’s misleading presentation and reported file-type discrepancy.

These findings justify containment, independent customer verification, and further investigation. They do not, on their own, establish authentication failure, confirmed malware execution, financial loss, or endpoint compromise.

## 13. Learning Outcomes

This investigation reinforces the importance of:

- Distinguishing display names, sender addresses, reply destinations, and envelope sender identities.
- Separating email authentication policies from message-specific authentication results.
- Inspecting file types rather than trusting filenames.
- Using hashes to identify exact file matches.
- Distinguishing infrastructure ownership from threat-actor attribution.
- Recording findings, interpretations, evidence gaps, and proposed actions separately.
