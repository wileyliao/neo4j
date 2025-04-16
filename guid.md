# Create Series
## Create a person named "Brie Larson", Birth "1989" </br>
`CREATE (a:Person {name:'Brie Larson', born:1989}) RETURN a`

## Create a movie(node), with following Properties: "titled": "Captain Marvel", "released": "2019", "Tagline": "Everything begins with a (her)o." </br>
`CREATE (a:Movie {title:'Captain Marvel', released:2019, tagline:'Everything begins with a (her)o.'}) RETURN a`

# Search Series
>- 如果不管edge的方向，則省略所有關係查找中的箭頭(">"or"<")
## Find a person named Tom Hanks </br>
`MATCH (a:Person {name:'Tom Hanks'}) RETURN a`

## Find persons names that start with Tom </br>
`MATCH (a:Person) WHERE a.name STARTS WITH 'Tom' RETURN a`

## Find movies released after 1990 but before 2000 </br>
`MATCH (a:Movie) WHERE a.released > 1990 AND a.released < 2000 RETURN a`

## Find all movies with actor "Tom Hanks" </br>
`MATCH (a:Person {name:'Tom Hanks'})-[:ACTED_IN]->(m:Movie) RETURN a,m`

## Find Who directed the movie "Cloud Atlas" </br>
`MATCH (m:Movie {title:'Cloud Atlas'})<-[:DIRECTED]-(d:Person) RETURN d`

## Find Co-actors of actor "Tom Hanks" </br>
`MATCH (a:Person {name:'Tom Hanks'})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(c) RETURN c.name`
>- a:Person {name:'Tom Hanks'} --> 找出演員 Tom Hanks
>- -[:ACTED_IN]->(m) --> 找出他演過的電影 m
>- <-[:ACTED_IN]-(c) --> 再找出也演過同一部電影的其他人（c）
>- RETURN c.name --> 回傳這些人的名字（共演者）
>>- 這樣會包含自己（Tom Hanks）本身，如果想排除，可以加：
>>- `WHERE c.name <> 'Tom Hanks'`
>>- <> means NOT EQUAL TO (!=)

## Find how people are related to movie "Cloud Atlas" </br>
`MATCH (people:Person)-[relatedTo]-(:Movie {title:'Cloud Atlas'})
RETURN people.name, type(relatedTo), relatedTo`
>- people:Person --> 所有的人物節點
>- -[relatedTo]- --> 與電影節點有任意方向、任意類型的關係
>- :Movie {title:'Cloud Atlas'} --> 電影節點，標題是 Cloud Atlas
>- RETURN people.name, type(relatedTo), relatedTo --> 回傳：
>>- people.name：這個人的名字
>>- type(relatedTo)：他與電影的關係類型（如 ACTED_IN, DIRECTED）
>>- relatedTo：這條關係的完整內容（包含屬性）

# Advance Search(Path: 路徑查詢 & hops: 跨多層關係查詢)
## Find movies and actors up to 4 "hops" away from Kevin Bacon
>- 找出距離 Kevin Bacon 1~4 步的電影或人（最多 4 hops）</br>
`MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..4]-(hollywood)
RETURN DISTINCT hollywood`
>- MATCH (bacon:Person {name: "Kevin Bacon"}) --> 找出 Kevin Bacon 的人物節點
>- -[*1..4]- --> 從他出發，沿著任意類型、任意方向的邊走 1 到 4 步（hops）
>- (hollywood) --> 每一個 hop 的終點節點，都記為 hollywood（可能是人、電影等）
>- RETURN DISTINCT hollywood --> 回傳所有找到的節點，不重複 </br>
### 一個 hop = 走過一條關係（edge），所以這條語法會找到：
1. Kevin Bacon 演的電影（1 hop）</br>
2. 跟他共演的人（2 hops）</br>
3. 再延伸出去的電影或人（最多到 4 hops）</br>

## Find The shortest path following any relationship from Kevin Bacon to "AI Pacino"
>- 找出 Kevin Bacon 到 Al Pacino 的最短路徑
`MATCH p=shortestPath(
              (bacon:Person {name:"Kevin Bacon"})-[*]-(a:Person {name:'Al Pacino'})
            )
RETURN p`
>- shortestPath(...) --> 尋找 Kevin Bacon 到 Al Pacino 的最短路徑（可能經過多部電影或其他演員）
>- -[*]- --> 沿著任意數量、任意類型、任意方向的關係走
>- p= --> 把整條路徑命名為 p
>- RETURN p --> 回傳這條最短路徑（包括中間經過的節點與邊）
>>- [*]	0 --> 個或多個 hops（沒有限制）
>>- [*1..4] --> 限制 hop 數在 1～4 之間（防止全圖爆炸）
>>- [*..6] --> 最多 6 hops
>>- [*3] --> 剛好 3 hops

# Delete and Update Series
## Delete all persons named "Brie Larson" </br>
`MATCH (a:Person {name:'Brie Larson'}) DETACH DELETE a`

## Delete all movies named "Captain Marvel" </br>
`MATCH (a:Movie {title:'Captain Marvel'}) DETACH DELETE a`

## Update a person named "Brie Larson", birth "1989" </br>
`MERGE (a:Person {name:'Brie Larson'})
ON CREATE SET a.born = 1989
ON MATCH SET a.stars = COALESCE(a.stars, 0) + 1
RETURN a`

>- 第一次執行時 → 建立一個 Person 節點，並設定 born = 1989，stars = 1
>- 第二次執行以後 → 不再建立新節點，而是讓 stars 自動加 1
>- MERGE	就像 SQL 的 UPSERT，若無則創建、若有則使用
>- ON CREATE SET --> 只有在創建新節點時才執行的設定
>- ON MATCH SET --> 只有在找到已存在的節點時才執行的設定
>- COALESCE(a.stars, 0)	若 a.stars 為空或不存在，就當作 0（避免錯誤）

## Person named "Brie Larson", act as "Carol Danvers" in movie titled "Captain Marvel" </br>
`MATCH (a:Person {name:'Brie Larson'}), (b:Movie {title:'Captain Marvel'})
MERGE (a)-[r:ACTED_IN]->(b) SET r.roles = ['Carol Danvers']
RETURN a,r,b`
>- MATCH (a:Person {name:'Brie Larson'}) --> 找出名字叫做 Brie Larson 的人（演員）節點，記為 a
>- MATCH (b:Movie {title:'Captain Marvel'}) --> 找出標題是 Captain Marvel 的電影節點，記為 b
>- MERGE (a)-[r:ACTED_IN]->(b) --> 建立或找到一條從 a 指向 b 的 ACTED_IN 關係，並記為 r
>- SET r.roles = ['Carol Danvers'] --> 在這條關係上，設定一個 roles 屬性為 ['Carol Danvers']（代表她在這部電影中的角色名）
>- RETURN a, r, b --> 回傳人、關係、電影節點三者的資料

## 進階使用
`MATCH (a:Person {name:'Brie Larson'}), (b:Movie {title:'Captain Marvel'})
MERGE (a)-[r:ACTED_IN]->(b)
SET r.roles = [x in r.roles WHERE x <> 'Captain Marvel'] + ['Captain Marvel']
RETURN a,r,b`
>- MATCH (a:Person {name: 'Brie Larson'}) --> 找出演員 Brie Larson 的節點
>- MATCH (b:Movie {title: 'Captain Marvel'}) --> 找出電影 Captain Marvel 的節點
>- MERGE (a)-[r:ACTED_IN]->(b) --> 如果 Brie Larson 和 Captain Marvel 之間尚未有 ACTED_IN 關係，就建立一個
>- SET r.roles = ... --> 設定或更新這條關係的 roles 陣列內容
>- [x IN r.roles WHERE x <> 'Captain Marvel'] + ['Captain Marvel'] --> 把原本的 r.roles 裡面，移除所有等於 'Captain Marvel' 的項目，再補上一個新的 'Captain Marvel'
>- RETURN a, r, b --> 回傳人、關係、電影三個資料項目 </br>
### 確保 'Captain Marvel' 這個角色只出現在 roles 一次 </br>
### 假設原本 r.roles = ['Captain Marvel', 'Captain Marvel'] </br>
### 執行後 r.roles = ['Captain Marvel'] </br>
### 假設原本r.roles = ['Carol Danvers'] </br>
### 執行後 r.roles = ['Carol Danvers', 'Captain Marvel'] </br>

