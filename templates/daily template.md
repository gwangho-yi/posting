---
date: <% tp.date.now() %>
achievement:
tags:
---


# Todo

- [ ] check box


# Time Tables
|  time   | do  |
| :-----: | :-: |
| 9:00 -  |     |
|         |     |
|         |     |
|         |     |
|         |     |
|         |     |

# Tommorow
- [ ] 


# Today Eat

| time | eat |
| :--: | :-: |
| 9:00 |     |
|      |     |
|      |     |
|      |     |
|      |     |
|      |     |



# Quote
<% tp.web.daily_quote() %>


<%* 
const currentMoment = moment(tp.file.title, "YYYY-MM-DD"); 
tR += '< '; tR += '[[' + currentMoment.format('YYYY|YYYY년') + ']]' + ' / '; tR += '[[' + currentMoment.format('YYYY-MM|MM월') + ']]' + ' / ';
tR += '[[' + currentMoment.format('gggg-[W]ww') + '|' + currentMoment.format('ww[주]') + ']]'; tR += ' >'; 
tR += '\n'; tR += '<< '; currentMoment.add(-1, 'days'); 
tR += '[[' + currentMoment.format('YYYY-MM-DD(ddd)') + ']]' + ' | '; currentMoment.add(1, 'days'); 
tR += currentMoment.format('YYYY-MM-DD(ddd)') + ' | '; currentMoment.add(1, 'days'); 
tR += '[[' + currentMoment.format('YYYY-MM-DD(ddd)') + ']]'; currentMoment.add(-1, 'days'); tR += ' >>'; 
%>