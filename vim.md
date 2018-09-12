Delete until character:
```
dt	<-- exclusive
df	<-- inclusive
```

Search and replace:
```
:%s/search/replace/g
```

Delete lines with match:
```
:g/match/d
:%g!/match/d	<-- delete lines without match
```

Copy one register into another:
```
:let @a=@b
```

Jump to character:
```
f<character>	<-- forward
F<character>	<-- backward
;		<-- repeat
,		<-- repeat backwards
```
