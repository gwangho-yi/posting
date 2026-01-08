---
date: <% tp.date.now() %>
achievement:
tags:
---


## Tasks
```tasks
(done today) OR (not done)
hide backlink

filter by function \
	const now = moment(); \
	return !task.file.filename.includes(now.format('YYYY-MM-DD'));
```

## TODO



## Tommorow


## Today Eat

| time | eat |
| :--: | :-: |
| 아침  |     |
|  점심  |     |
|  저녁  |     |
|      |     |
|      |     |
|      |     |



## Quote
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