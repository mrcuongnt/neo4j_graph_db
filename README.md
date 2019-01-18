# neo4j_graph_db
Tìm hiểu cơ sở dữ liệu đồ thị Neo4j
Trong phần này, chúng ta sẽ thực hiện các thao tác trên cơ sở dữ liệu Neo4j và My SQL  
Chuẩn bị dữ liệu để import vào Database
Mô hình dữ liệu: Database lưu trữ thông tin phim ảnh và các thông tin liên quan
  
•	Danh sách Node:
o	Person 
o	Movie
o	Genre
o	Keyword
•	Danh sách Relationship
o	Movie-[:HAS_GENRE]->Genre
o	Movie-[HAS_KEYWORD]->Keyword
o	Person-[WRITER_OF]->Movie
o	Person-[PRODUCED]->Movie
o	Person-[DIRECTED]->Movie
o	Person-[ACTED_IN]->Movie
File import: các file dữ liệu import chuẩn trong file neo4j-movies-template-master.zip, thư mục csv. Các dữ liệu sinh ngẫu nhiên bằng cách chạy chương trình java: GenData.jar
Thực hiện tạo bảng trên Database My SQL và import bằng tool ETL (Không đề cập chi tiết trong báo cáo này)
Thực hiện import trên Database Neo4j
Lưu ý copy toàn bộ file cần import vào đường dẫn trước khi thực hiện: {neo4j_home_install}/import
Thực hiện chạy lần lượt các lệnh trong file setupCypherCommands.txt (chi tiết lệnh chạy và thời gian chạy demo các lệnh trong file này)
Kết quả một số truy vấn trên Neo4j và My SQL
Truy vấn	My SQL	NEO4J
Import MOVIE 	Import bảng MOVIE bằng tool ETL	USING PERIODIC COMMIT 3000
LOAD CSV WITH HEADERS FROM "file:///movie_node_fake.csv" AS r FIELDTERMINATOR ';'
CREATE (m:Movie {
  id: toInteger(r.`id:ID(Movie)`),
  title: r.title,
  duration: toInteger(r.`duration:int`),
  rated: r.rated
});
		Added 30000000 labels, created 30000000 nodes, set 90000000 properties, completed after 1556438 ms
Import PERSON 	Import bảng PERSON bằng tool ETL	USING PERIODIC COMMIT 3000
LOAD CSV WITH HEADERS FROM "file:///person_node_fake.csv" AS r FIELDTERMINATOR ';'
CREATE (p:Person {
  id: toInteger(r.`id:ID(Person)`),
  name: r.name,
  born: toInteger(r.`born:int`)
});
		Added 29999999 labels, created 29999999 nodes, set 89999997 properties, completed after 1134096 ms.
Import quan hệ Movie has keyword	Bảng movie có trường keyword_id	USING PERIODIC COMMIT 3000
LOAD CSV WITH HEADERS FROM "file:///has_keyword_rels_fake.csv" AS r FIELDTERMINATOR ';'
MATCH (m:Movie {id: toInteger(r.`:START_ID(Movie)`)}), (k:Keyword {id: toInteger(r.`:END_ID(Keyword)`)})
CREATE (m)-[:HAS_KEYWORD]->(k);
		Created 24431359 relationships, completed after 1240171 ms.
Import quan hệ Person acted_in Movie	Import bảng PERSON_ACTED_IN_MOVIE_REL bằng tool ETL	USING PERIODIC COMMIT 3000
LOAD CSV WITH HEADERS FROM "file:///has_keyword_rels_fake.csv" AS r FIELDTERMINATOR ';'
MATCH (m:Movie {id: toInteger(r.`:START_ID(Movie)`)}), (k:Keyword {id: toInteger(r.`:END_ID(Keyword)`)})
CREATE (m)-[:HAS_KEYWORD]->(k);
		Set 29999995 properties, created 29999995 relationships, completed after 2942974 ms.
Tạo index	CREATE INDEX movie_title_idx ON movie(title);	CREATE INDEX ON :Movie(title);
//Added 1 index, completed after 30 ms.
Với Movie.title = Elysium, tìm các phim khác cùng keyword với phim này	Select * from movie where keyword_id in (select keyword_id from movie where title = 'Elysium') and title <> 'Elysium' limit 25	MATCH (m:Movie)
WHERE  m.title = 'Elysium'
MATCH (m)-[:HAS_KEYWORD]->(k:Keyword)
MATCH (movie:Movie)-[r:HAS_KEYWORD]->(k)
WHERE m <> movie 
WITH movie, k RETURN movie, k
limit 25
	0.8s	Started streaming 25 records after 2 ms and completed after 272 ms.

Với Movie.title = Elysium, tìm các phim khác cùng diễn viên với phim này	Select distinct movie.* from movie m, person_acted_in_movie_rel pm, person_acted_in_movie_rel pm2, movie movie 
where m.id = pm.movie_id 
and pm.person_id = pm2.person_id
and pm2.movie_id = movie.id
and m.title = 'Elysium' 
and movie.title <> 'Elysium' 
limit 25	MATCH (m:Movie)
WHERE  m.title = 'Elysium'
MATCH (m)<-[:ACTED_IN]-(a:Person)
MATCH (a)-[r:ACTED_IN]->(movie:Movie)
WHERE m <> movie
WITH movie
RETURN movie
LIMIT 25
	1.3s	Started streaming 3 records after 2 ms and completed after 546 ms.
