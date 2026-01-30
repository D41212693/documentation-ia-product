---
title: Release Notes
description: Descartes R5 Release Notes
---

# Release Notes Descartes R5

## Version Descartes R5 SP8

### Bug fixes

- **IGRC-4855** some trawpermlinkreview are not renewed on next timeslot

## Version Descartes R5 SP7

### New Features

- **IGRC-4847:** Added key attributes for count optimization and JSON aggregation on join components
- **BWIPUAR-2262:** Launch tech workflows post data upload

### Bug fixes

- **IGRC-4844:** When using LIMIT and OFFSET with postgres, the query can be very slow
- **COL-1605:** Workflow task expiration date is different if escalation reminder is set or not
- **COL-1618:** Reminder dates set on a non-business day when setting up access review campaign
- **COL-1659:** Renew reconciliations can get stuck. You can now independently disable history searches during reconciliation renewals with new Java options. This helps prevent issues where renewals get stuck.
  - `-Ddisablereconsearchinhistoryforleavers=true`
    Disable the search of identities in history when renewing a 'leave' reconciliation.
    When a 'leave' reconciliation exists in the previous timeslot, the product tries to find the latest reconciliation in the history to find the identity and renew this one if the identity exists in the current timeslot.
  - `-Ddisablereconsearchinhistoryfororphans=true`
    Disable the search of reconciliation in history when renewing an 'orphan' reconciliation.
    When no reconciliation is found for an account in the previous timeslot, the product tries to find the latest reconciliation in the history to renew it (either identity or service reconciliation) in the current timeslot.
- **COL-1675:** Portal freeze after launching background tasks
- **COL-1693:** Anonymous Birt reports don't work on Descartes R5 SP5

## Version Descartes R5 SP6

### Bug fixes

- **IGRC-4839:** Fix datetime widget bug in Pages DateTime, parameters and picker
- **IGRC-4840:** Mashup parameters does not crash anymore
- **IGRC-4842:** Manually set managers are not reported from timeslot to timeslot IF there is not manager policy in the project
- **BWIPUAR-2205:** pass current timeslot to workflow in react
- **BWIPUAR-2231:** fix dashboard edit button disappears when cancelling project dashboard edition
- **BWIPUAR-2232:** fix crosstab css conflicting with `primeflex.css`
- **BWIPUAR-2256:** sticky timeslot: add sticktimeslot property to Page tag
- **BWIPUAR-2269:** Avoid the purge of a timeslot if used by a attached to timeslot review
- **BWIPUAR-2301:** clicking on checkbox is slow in large dataset tables
- **COL-1669:** BusinessViews' timeslot param incorrectly causes the view to run on the latest timeslot

## Version Descartes R5 SP5

### Bug fixes

- **IGRC-4841:** Manually set managers (other than identity) are not reported from timeslot to timeslot

## Version Descartes R5 SP4

### Security fix

- **IGRC-4836:** Remove common-compress library located in native xlsx emitter (security fix)

## Version Descartes R5 SP3

### New Features

- **IGRC-4834:** Allow custom AD repository types in AD accounts activation
- **BWIPUAR-2205:** pass current timeslot to workflow

### Bug fixes

- **IGRC-4831:** Metadata created on a past timeslot are not written correctly in database

## Version Descartes R5 SP2

### Bug fixes

- **IGRC-4827:** indirect manager links are copied from previous timeslot
- **IGRC-4828:** Reuse the same global classloader upon several Business View executions
- **COL-1641:** Leave date of leaver account updated if left identity does not have departure date

## Version Descartes R5 SP1

### New Features

- **COL-1590:** Log memory/GC infos in batch
- **COL-1610:** Log batch options used

### Bug fixes

- **IGRC-4780:** Impossible to get the grace period date in notification
- **IGRC-4814:** Missing translation key `deleteImportTheoricalAccRights`
- **IGRC-4817:** TypedQuery error in the exception tab of controls in the studio
- **IGRC-4825:** IAM theoretical rights can crash with duplicates
- **COL-1527:** Studio freeze when merging review wizard generate files
- **COL-1591:** Ticket review update slow when updating
- **COL-1606:** Workflow - Parameter mapping in NotifyRule show Undefined for the mapped value
- **COL-1625:** A new classloader is created for each script executed, inducing a memory leak
- **COL-1631:** TaskComplete wait-end latency
- **COL-1639:** IAM right target error during collecting phase

## Version Descartes R5

### Bug fixes

- This is a security fix with an upgrade of the whole Eclipse platform
