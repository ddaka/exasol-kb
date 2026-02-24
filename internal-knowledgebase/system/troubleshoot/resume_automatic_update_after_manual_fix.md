---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Internal Knowledgebase: Resume Automatic Update After Manual Fix"
summary: "The following guide will help the support team resume the automatic upgrade process after a manual fix has been applied. An alert will be triggered if the update fails during an..."
---
# Internal Knowledgebase: Resume Automatic Update After Manual Fix

The following guide will help the support team resume the automatic upgrade process after a manual fix has been applied. An alert will be triggered if the update fails during an attempt to upgrade the database.
### Automatic Update Failure Alert
**Sample Alert Message:**
```json
{
  "Automatic_Upgrade_Failed-RunTheseCommandsManually": {
    "AFFECTED-DB": "ZDiy3w_xTAC4ZqxWFDqzbQ",
    "ResumeCommand": "aws stepfunctions send-task-success --task-token AQCMAAAAKgAAAAMAAAAAAAAAAbkfPIKPlok5+m+DRZkyUZrSlfK/yAnt6dOtILfyTzzW2uDk8sE2u8A+KcQp6Oapliw1XzjnFPha0St9z12ImSqA7s9rXG8avTekHVGUq1uOCcgKLEcoWg==tQT75ix8a/iml2Rv0XuhgJtlx80MYwwbHAKFY9gXRh17OEjM4UcJdFNF4VUitV6iec3KwGYPQIbv14C7ItHWkJcHuynnEU1TnFOpK2JDWRFotslZ6o0sX3TkLJ2IfBSFS3CJ1rFYsk0lV0NaS++nfuYdLg2fYwN+JjpcdbYUEJ1hSUc/TbMILTddARRdsJEN2d48n2n+OU2LWJQJFY2r7bfQZaITKJ2c86neylK59lhSV6wGXAN/Wpv5jo5F4jmmYmbp6AhSSOjFo5ByT4SKX+NjBTnY6z2rqiTt79v74b2FaiAcLOeybzBY/wYGJkACi4dVh/3MmDsBC6ub9VEsyRmzeuala4S2Fhp0gorbfu3ycL+Gjr+voie6jajqLZRp3z9kEzN2nwNIufIME9B353odpx/jb+RAJ5Bw+z6wgwqClQ7fADP9cuafVi6zyiUXdmCkIhwgEyqE9uLfTuzLnuUAEDFeF19QEgMyWVeRc4Rd8QeT77D9j9F55De0+u4yfSAsMuMCykCpeGscvCjp --task-output {} --profile dev-saas"
  }
}
```
**Resume Automatic Upgrade workflow:**
* Run the ResumeCommand provided in the alert above from your local terminal:
```bash
    aws stepfunctions send-task-success --task-token AQCMAAAAKgAAAAMAAAAAAAAAAbkfPIKPlok5+m+DRZkyUZrSlfK/yAnt6dOtILfyTzzW2uDk8sE2u8A=+KcQp6Oapliw1XzjnFPha0St9z12ImSqA7s9rXG8avTekHVGUq1uOCcgKLEcoWg==tQT75ix8aiml2Rv0XuhgJtlx80MYwwbHAKFY9gXRh17OEjM4UcJdFNF4VUitV6iec3KwGYPQIbv14C7ItHWkJcHuynnEU1TnFOpK2JDWRFotslZ6o0sX3TkLJ2IfBSFS3CJ1rFYsk0lV0NaS++nfuYdLg2fYwN+JjpcdbYUEJ1hSUc/TbMILTddARRdsJEN2d48n2n+OU2LWJQJFY2r7bfQZaITKJ2c86neylK59lhSV6wGXAN/Wpv5jo5F4jmmYmbp6AhSSOjFo5ByT4SKX+NjBTnY6z2rqiTt79v74b2FaiAcLOeybzBY/wYGJkACi4dVh/3MmDsBC6ub9VEsyRmzeuala4S2Fhp0gorbfu3ycL+Gjr+voie6jajqLZRp3z9kEzN2nwNIufIME9B353odpx/jb+RAJ5Bw+z6wgwqClQ7fADP9cuafVi6zyiUXdmCkIhwgEyqE9uLfTuzLnuUAEDFeF19QEgMyWVeRc4Rd8QeT77D9j9F55De0+u4yfSAsMuMCykCpeGscvCjp --task-output {} --profile dev-saas
```
* Make sure to set the correct profile name under the --profile argument according to your local configuration in ~/.aws/config (e.g., dev-saas).
***Note: Contact the Product Engineering Team if the issue has not been resolved.***
