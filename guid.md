Find a person named Tom Hanks </br>
`MATCH (a:Person {name:'Tom Hanks'}) RETURN a`

Create a person named "Brie Larson", Birth "1989" </br>
`CREATE (a:Person {name:'Brie Larson', born:1989}) RETURN a`

Create a movie(node), with following Properties: "titled": "Captain Marvel", "released": "2019", "Tagline": "Everything begins with a (her)o." </br>
`CREATE (a:Movie {title:'Captain Marvel', released:2019, tagline:'Everything begins with a (her)o.'}) RETURN a`

Delete all persons named "Brie Larson" </br>
`MATCH (a:Person {name:'Brie Larson'}) DETACH DELETE a`

Delete all movies named "Captain Marvel" </br>
`MATCH (a:Movie {title:'Captain Marvel'}) DETACH DELETE a`

Update a person named "Brie Larson", birth "1989" </br>
-< 第一次執行時 → 建立一個 Person 節點，並設定 born = 1989，stars = 1
-< 第二次執行以後 → 不再建立新節點，而是讓 stars 自動加 1
-< MERGE	就像 SQL 的 UPSERT，若無則創建、若有則使用
-< ON CREATE SET --> 只有在創建新節點時才執行的設定
-< ON MATCH SET --> 只有在找到已存在的節點時才執行的設定
-< COALESCE(a.stars, 0)	若 a.stars 為空或不存在，就當作 0（避免錯誤）
`MERGE (a:Person {name:'Brie Larson'})
ON CREATE SET a.born = 1989
ON MATCH SET a.stars = COALESCE(a.stars, 0) + 1
RETURN a`

