## Create ATC node
`MERGE (:ATC {code:"N02AX02"})`
>- 如果圖中已經有 :ATC {code: "N02AX02"}，就不會重複建立
>- 如果沒有，就會建立一個新的節點

`CREATE CONSTRAINT unique_atc_code
FOR (a:ATC)
REQUIRE a.code IS UNIQUE`
>- 確保 ATC 節點的 code 欄位永遠不會重複
>- 查看：`MATCH (a:ATC {code: "N02AX02"})
RETURN a`

## Create ingredient node
`CREATE CONSTRAINT unique_ingredient_name
FOR (i:Ingredient)
REQUIRE i.name IS UNIQUE`
>- 讓 Ingredient 的 name 成為唯一識別，不會重複。
```
MERGE (i1:Ingredient {name: "TRAMADOL"})
SET i1.ddd = 0.3, i1.unit = "g"

MERGE (i2:Ingredient {name: "PARACETAMOL"})
SET i2.ddd = 3, i2.unit = "g"
```
>- 改大小寫：
`MATCH (i:Ingredient)
REMOVE i:Ingredient
SET i:INGREDIENT`
