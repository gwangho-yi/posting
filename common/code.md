
# 데일리 템플릿 페이징 코드
```javascript
<%* 
const currentMoment = moment(tp.file.title, "YYYY-MM-DD"); 
tR += '<'; tR += '[[' + currentMoment.format('YYYY|YYYY년') + ']]' + ' / '; tR += '[[' + currentMoment.format('YYYY-MM|MM월') + ']]' + ' / ';
tR += '[[' + currentMoment.format('gggg-[W]ww') + '|' + currentMoment.format('ww[주]') + ']]'; tR += ' >'; 
tR += '\n'; tR += '<< '; currentMoment.add(-1, 'days'); 
tR += '[[' + currentMoment.format('YYYY-MM-DD(ddd)') + ']]' + '|'; currentMoment.add(1, 'days'); 
tR += currentMoment.format('YYYY-MM-DD(ddd)') + ' | '; currentMoment.add(1, 'days'); 
tR += '[[' + currentMoment.format('YYYY-MM-DD(ddd)') + ']]'; currentMoment.add(-1, 'days'); tR += ' >>'; 
%>
```