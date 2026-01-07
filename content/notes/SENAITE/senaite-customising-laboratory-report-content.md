+++
title = "SENAITE: customising laboratory report content"
+++

[senaite/senaite.patient: Patient handling for SENAITE](https://github.com/senaite/senaite.patient) is the add-on that adds the capability to SENAITE to manage patient data but unfortunately it does not include any patient information in lab reports (COA, or Certificates Of Analysis in SENAITE parlance) by default. The COA templates used are in the folder `/home/senaite/buildout-cache/eggs/cp27mu/senaite.impress-2.5.0-py2.7.egg/senaite/impress/templates/reports/` - you can create a Plone add-on which overrides these templates (hard if you don't know Plone) or simply edit the templates in place (easier but your changes will disappear if you upgrade `senaite.patient`). Plone is pretty complex and it may be a while before I am creating my own add-ons, so I went with the easy option.

The COA templates are written in something called [TALES](https://en.wikipedia.org/wiki/Template_Attribute_Language), or Template Attribute Language. This consists of HTML elements (which do not change) and XML elements (which reflect the dynamic parts) - it is a bit like RMarkdown, where the Markdown does not change but the R code chunks are converted to something else when the document is created. Here Python does the converting - you can even reference Python functions or use Python code. Beyond that I don't claim to understand TALES much yet, but with a little bit of HTML knowledge and the help of the convenient [SENAITE SuperModel](https://github.com/senaite/senaite.app.supermodel) I was able to create my own COA template from one of the existing ones (`MultiDefault.pt`) to include the lab and patient information I wanted, as well as the logo of the lab.  

- Including `<tal:report define="model python:view.model;">` at the start indicates that I want to use the SENAITE SuperModel - this is a "content wrapper" which allows you to reference data through a unified interface. 
- You can see this in use with `<a tal:content="model/getPatientFullName"/>` - I found the names of the elements by rooting through the source code (there is probably a better way).
- With a bit of trial and error and fiddling with the CSS I changed the header of the report, as in the section beginning `NEW REPORT HEADER`. I had already uploaded the logo image file using the instructions here: [1.4.1 Replace the LIMS logo, banner, title and favicon — LIMS Collective](https://www.bikalims.org/manual/introduction-and-overview/client-facing-lab-portal/replace-you-lims-banner)
- The section beginning `NEW PATIENT INFO` is the main other bit I added, showing patient information in a tabular format. To format Age I used a bit of Python code; to format Sex I used a bit of TALES logic. 

The result looks good. A generic version of my final code is shown below.

```html
<tal:report define="model python:view.model;">

  <!-- REPORT CONTROLS -->
  <tal:t replace="structure python:view.render_controls(context, **options)" />

  <!-- REPORT JS -->
  <tal:t replace="structure python:view.render_js(context, **options)" />

  <!-- REPORT CSS -->
  <tal:t replace="structure python:view.render_css(context, **options)" />

  <!-- NEW REPORT HEADER -->
  <!-- Header Table -->
  <div class="row section-header no-gutters">
  <!-- Header Left -->
  <div class="col-2 text-left">
  <img class="logo image-fluid" style="object-fit:contain; height:60px; margin-top:0px; margin-left:15px" src="https://senaite.example.com/senaite/your_logo.jpg" alt="Logo" title="Logo">
  </img>
  </div>
  <div class="col-10 text-left">
  <!-- Header Right -->
  <h1><strong>Name of Laboratory</strong></h1>
  <h1>Analysis Report</h1>
  </div>
  </div>

  <!-- REPORT INFO -->
  <tal:t replace="structure python:view.render_info(context, **options)" />

  <!-- REPORT ALERTS -->
  <tal:t replace="structure python:view.render_alerts(context, **options)" />

  <!-- NEW PATIENT INFO -->
    <div class="row section-summary no-gutters">
  <div class="w-100">
  <h1>Patient Information</h1>
  <table class="table table-sm table-condensed">
  <tr>
  <td class="label" i18n:translate="">
    <div>Patient MRN</div>
  </td>
  <td>
    <a tal:on-error="nothing" tal:content="model/MedicalRecordNumber/value"/>
  </td>
  </tr>
  <tr>
  <td class="label" i18n:translate="">
    <div>Patient Full Name</div>
  </td>
  <td>
    <a tal:on-error="nothing" tal:content="model/getPatientFullName"/>
  </td>
  </tr>
  <tr>
  <td class="label" i18n:translate="">
  <div>Age</div>
  </td>
  <td>
  <a tal:on-error="nothing" tal:content="python: '%s years, %s months' % (model.getAge().years, model.getAge().months)"/>
  </td>
  </tr>
  <tr>
  <td class="label" i18n:translate="">
    <div>Patient Date Of Birth</div>
  </td>
  <td>
    <a tal:on-error="nothing" tal:content="python: model.getDateOfBirth()[0].strftime('%d/%m/%Y')" />
    <a tal:on-error="nothing" tal:condition="python: model.getDateOfBirthEstimated()" tal:content="python: '(NB: estimated from age)'"/>
  </td>
  </tr>
  <tr>
  <td class="label" i18n:translate="">
    <div>Patient Sex</div>
  </td>
  <td>
    <a tal:condition="python: model.Sex == 'm'" tal:content="python: 'Male'" />
    <a tal:condition="python: model.Sex == 'f'" tal:content="python: 'Female'" />
    <a tal:condition="python: model.Sex == ''" tal:content="python: 'Unspecified'" />
  </td>
  </tr>
  <tr>
  <td class="label" i18n:translate="">
    <div>Patient Address</div>
  </td>
  <td>
    <a tal:on-error="nothing" tal:content="model/PatientAddress"/>
  </td>
  </tr>
  </table>
  </div>
  </div>

  <!-- REPORT RESULTS -->
  <tal:t replace="structure python:view.render_results(context, **options)" />

  <!-- RESULTS INTERPRETATION -->
  <tal:t replace="structure python:view.render_interpretations(context, **options)" />

  <!-- SAMPLE REMARKS -->
  <tal:t replace="structure python:view.render_remarks(context, **options)" />

  <!-- REPORT SUMMARY -->
  <tal:t replace="structure python:view.render_summary(context, **options)" />

  <!-- REPORT ATTACHMENTS -->
  <tal:t replace="structure python:view.render_attachments(context, **options)" />

  <!-- REPORT SIGNATURES -->
  <tal:t replace="structure python:view.render_signatures(context, **options)" />

  <!-- REPORT DISCREETER -->
  <tal:t replace="structure python:view.render_discreeter(context, **options)" />

  <!-- REPORT FOOTER -->
  <tal:t replace="structure python:view.render_footer(context, **options)" />

 <div style="font-size: 0.8em; color: #555; line-height: 1.4; margin-top: 20px; border-top: 1px solid #ccc; padding-top: 10px;">
      <strong>Disclaimer:</strong> This laboratory report is intended for use by healthcare professionals. All results should be interpreted in conjunction with the patient's clinical history, symptoms, and other diagnostic information.
      <br><br>
      <strong>Limitations and Accuracy:</strong> While every effort is made to ensure the accuracy of test results, laboratory testing is subject to potential <strong>pre-analytical, analytical, and post-analytical errors</strong>. These may include sample collection or handling issues, analytical variability, or reporting inaccuracies.
      <br><br>
      <strong>False Results:</strong> The presence of false positive or false negative results is a recognized limitation of laboratory testing. Such occurrences are inherent risks of complex diagnostic procedures and should not be interpreted as negligence.
      <br><br>
      <strong>Repeat Testing:</strong> If test results are unclear, unexpected, or inconclusive, a repeat test can be requested within <strong>24 hours</strong> at no additional cost. Repeat testing requests must be accompanied by relevant clinical notes and details of the clinical ambiguity or suspected discordance.
      <br><br>
      <strong>Clinical Correlation:</strong> Laboratory results are only part of the diagnostic process. They must be interpreted alongside the patient's clinical presentation, medical history, and other diagnostic investigations.
      <br><br>
      <strong>Confidentiality and Data Protection:</strong> This report contains confidential patient information. It is intended solely for the authorized recipient. Unauthorized use, disclosure, or distribution is prohibited.
  </div>
	
</tal:report>
```
