# Interview Guide

YouTrack: [DIP-5](https://medexchange-diploma.youtrack.cloud/issue/DIP-5) · Owner: Almasuly Damir (interviews together with Korazbay Alikhan)

## Purpose

Confirm or reject the problem statement of the [proposal](../01-proposal.md) and collect evidence for the functional requirements in [06-requirements.md](../06-requirements.md). Each finding is recorded in [interviews.md](interviews.md) and linked to the requirements it supports.

## Participants

| Group | Who | Target number |
|---|---|---|
| Medical staff | Doctors, registrars / medical records staff who send or receive documents about patients | 2–3 |
| IT / information security | IT specialist or information security officer of a medical organization | 1–2 |

## Rules

- 20–30 minutes per interview, in person or by video call; Russian or Kazakh.
- Ask for consent to take notes before starting. No recording without explicit consent.
- Do not collect any patient data. If a participant shows a real document, do not photograph it.
- Ask about past behavior ("Tell me about the last time…"), not opinions about our idea. Show the prototype only at the end.
- Record the participant only by role and type of organization (for example, "P1, therapist, city polyclinic"); names stay with the interviewer.
- Mark each finding as a direct statement of the participant or as the interviewer's interpretation.

## Introduction (2 min)

> We are students of [university] working on a diploma project about the exchange of medical documents between organizations. We want to understand how this works for you today. There are no right or wrong answers, and we will not record your name or any patient data. May I take notes?

## Questions for medical staff

**Context**

1. What is your role, and what type of organization do you work in?
2. Which documents about patients do you send to or receive from other organizations? (referrals, discharge summaries, test results, certificates, extracts)
3. Roughly how often does this happen per week?

**Current process**

4. Tell me about the last time you had to send a document to another organization. What did you do, step by step?
5. Which channels do you use: a medical information system (Damumed, KMIS…), e-mail, messengers, paper, courier, fax?
6. How do you get documents from other organizations? Where do they arrive, and who sorts them?
7. How do you know that the recipient received and opened the document?

**Problems**

8. What was the last problem with such an exchange? (lost document, wrong recipient, delay, unreadable scan, missing signature)
9. Has a document ever been sent to the wrong person or organization? What happened?
10. How do you check that a received document is genuine and was not changed?
11. How much time per week do you spend on printing, scanning, calls and re-sending?

**Signature and consent**

12. Do you sign documents with an EDS (NCALayer)? For which documents? Which problems do you have with it?
13. Do you need the patient's consent before sending their documents? How is it recorded?

**Expectations**

14. If you could change one thing in this process, what would it be?
15. Which information about a document do you need to see at a glance? (patient, type, sender, status, signature)
16. *(optional, at the end)* Show the prototype: "Try to send a discharge summary to the cardiology center." Observe without help and note where the participant hesitates.

## Questions for IT / information security

1. Which systems does your organization use for exchanging documents with other organizations?
2. Who grants and revokes users' access? How long does it take when an employee leaves?
3. Do you have a log of who opened or sent a document? Have you ever needed it for an incident?
4. Which requirements do you have to meet for personal and medical data (Law No. 94-V, internal policies, audits)?
5. How are EDS keys distributed to employees and stored (file, token, ID card)?
6. What would prevent your organization from connecting a new exchange system? (integration with the MIS, security approval, cost, training)
7. Which authentication do you expect: password, two-factor, EDS login, single sign-on?

## Closing (1 min)

- "Is there anything about this process that we did not ask but should have?"
- Ask whether they can be contacted again for a usability test of the prototype.

## After the interview

Within 24 hours, add the record to [interviews.md](interviews.md) using its template, and update the *Source* and *Status* columns of the affected requirements in [06-requirements.md](../06-requirements.md).
