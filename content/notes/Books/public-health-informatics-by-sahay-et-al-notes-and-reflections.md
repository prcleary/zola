+++
title = "Public Health Informatics by Sahay et al - notes and reflections"
+++

![](https://global.oup.com/academic/covers/pdp/9780198758778)


{% aside(position="right") %}
Public Health Informatics: Designing for change - a developing country perspective 

Sundeep Sahay, T Sundararaman, and Jørn Braa

Published: 2017

ISBN: 9780198758778

[Publisher link](https://global.oup.com/academic/product/public-health-informatics-9780198758778)
{% end %}

This is essential reading for anyone involved in developing public health information systems in low and middle-income countries. It is a synthesis of decades of broad experience, with several case studies, and would make an excellent source text for a university module on the subject. Two of the three authors are associated with the [DHIS 2](@/notes/DHIS2/_index.md) software platform, but in keeping with a key theme of this book that the major challenges are often non-technical, there is relatively little discussion of specific technical solutions such as DHIS 2. In fact this is not a technical informatics book per se, and so would be of value to a broad readership including policy makers, programme managers and academics.

The authors begin with a historical perspective on developments in relevant areas such as computer science and health informatics, and end with calls to action to adopt an "expanded public health informatics" framework to meet the many information needs of goals such as universal health coverage in LMIC, where hospital data (focussing on individual patients) would be integrated with public health data (focussing on aggregate indicators) and relevant data from other sources. In the main part of the book they review the many challenges (and the many previous failed attempts) in public health informatics and the learning for future such efforts, often bringing in conceptual frameworks from disciplines such as organisational theory. 

I recognised many of the challenges and frustrations from my own experience and occasionally chuckled at their optimism-with-scepticism attitude:

> "Typically, this [*the information systems lifecycle*] starts from expressing requirements, procuring a service provider to build a system, heightening of expectations of how digitization will solve a multitude of problems, and then a period of implementation, a readjustment of expectations, growing frustration and criticisms and mutual recrimination, leading to abandonment or obsolescence of the system, and its replacement through the next cycle."

Any work to successfully strengthen information systems in LMIC will need to engage with the complexity of numerous issues including: fragmentation; legacy systems lacking interoperability; a range of internal and external stakeholders with conflicting agendas; lack of resource or skills; the lag between technical and legislative developments; poor leadership and weak governance; procurement restrictions (limiting open source software options); poor data quality (sometimes related to perverse incentives); absent common data standards; lack of information security awareness; and limited analysis and use of existing data. 

There is no simple or quick or common solution to all of this but principles can be derived from previous experience and the research literature on the approaches that do or do not tend to work. 

Centralised, top-down approaches have a poor record, as do outsourced/vendor-specific solutions or purely technically-focussed approaches. 
Centrally-defined data sets with a punitive approach to poor data quality and limited value of the data to lower level users also don't generally work well. 
Computerisation of unstandardised health information (such as elements of personal health records) is difficult without a prior process of formalisation. 

Where modest success has been achieved in a complex environment, it has more often been via a bottom-up, participatory, incrementalist, evolutionary approach (which can take time), or through the introduction of an "attractor for change" (an element such as a common dashboard or common standard that is valued by different stakeholders and incentivises adoption and collaboration rather than imposing a specific solution). Better use of data will drive improvements in data quality more than sanctions will. Open source solutions (DHIS 2 and OpenMRS are mentioned) often work well with an evolutionary, participatory approach, though may sometimes be disadvantaged by government procurement rules; they also require acquisition of specific technical skills (e.g. for maintaining Linux virtual machines in the cloud or a data centre) for full ownership of the system without on-going reliance on external technical expertise. 

I think this book will change and improve my practice. Over my six short years of experience in public health informatics, I had arrived at some of the same conclusions by experience, intuition or accident, but this book provides authoritative confirmation of some aspects of my thinking, while also highlighting neglected areas and opening up my narrow-minded focus on IHR to the wider information needs of LMIC. 

I was converted to open source software a long time ago, which likely strengthened my conviction that we should use DHIS 2, at a time when DHIS 2 was not the leading solution it is now. Advocacy is a delayed-effect process and difficult to attribute. I have lost count of the number of meetings I have had with colleagues from different provinces, sectors or others over the years, advocating an open source, integrated, sustainable, secure approach to an often sceptical audience and seeming to get nowhere. But just when you think you have achieved nothing despite saying the same things dozens of times, you start to hear similar messages emanating from partners, or hear of positive developments in line with your urgings. You don't know if it is because of you, but it is still a good feeling. I also think that to some extent, our implementation of DHIS 2 became an "attractor for change" as people started to see everything that it provided "out of the box", and compared it to their own in-house/bespoke solutions where they existed. 

I was very narrowly focussed on the technical aspects initially (with the excuse that I was the main person required to do them) but I have increasingly recognised the need for involving and giving value to all levels of the system, and the need for communities of practice and conversations about data. We need to do a lot more to demonstrate the value of the data to all participants, particularly at the lower levels. 

I was sometimes frustrated by the slow pace of change (with occasional setbacks) and the defensive attitude of vested interests, and often feared that my gradualist approach (dictated by limited resource) was not the "proper" way to do things. This book has given me the encouragement that in the complex environment where I work, a grand strategic top-down centrally-imposed implementation would not necessarily have worked, and what I saw as frustrations were actually a necessary process of negotiation. 

The book also has some interesting thinking on data governance and policy, and how this relates to use of the cloud - I couldn't initially understand why we couldn't use cheap commercial cloud services, but I now understand that even there is no explicit prohibition there may be strongly held implicit views on data ownership related to storage location. 

The book does address information security in a number of places but I think this is a very important area which could benefit from expansion to a separate chapter in a later edition. 

The authors also recommend greater use of research methods to inform projects and share learning; I do have pangs of guilt that despite much experience we have not yet added anything of note to the research literature. I feel more motivated to add things to this blog than to write papers. 

This is a well-presented and structured book. There are a number of typoes and the occasional non-native English construction, but in general it is an easyish read after the early more theoretical chapters. I only noted one technical error; there seems to be some confusion between SSH and SSDs on page 108. 


