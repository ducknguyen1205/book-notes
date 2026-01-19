- **Objective:** To perform full-text searches within a database.

- **Antipattern:** Utilizing pattern-matching predicates (such as `LIKE` or `REGEXP`) for full-text search.

- This antipattern involves using `LIKE` or regular expressions (`REGEXP`) to find words within text columns.
- **Example:**
	
	```sql
	SELECT * FROM Bugs WHERE description LIKE '%crash%';
	```

The previous example matches text that contains the words one, but it also matches strings money, prone, lonely, and so on. Searching for a pattern with the key word delimited by spaces doesn’t match occurrences of the word with punctuation or at the start or end of the text. The regular expressions supported by your database might support a special pattern for a word boundary:

```sql
SELECT * FROM Bugs WHERE description REGEXP '[[:<:]]one[[:>:]]';
```

- **How to Recognize:** Look for SQL queries using `LIKE` with wildcards (e.g., `'%keyword%'`) or regular expressions to search for text.
- **Legitimate Uses:** While generally an antipattern for full-text search, it can be acceptable for very infrequent queries or in situations where the dataset is known to be small.
- **Solution:** Employ the appropriate tools for the job. This often means using features specifically designed for full-text searching:

- **Vendor Extensions:** Most major database systems offer built-in full-text search capabilities.
	
	- _MySQL:_ Offers `FULLTEXT` indexes and the `MATCH()` function (for `MyISAM` storage engine).
		
		```sql
		ALTER TABLE Bugs ADD FULLTEXT INDEX bugfts (summary, description);
		SELECT * FROM Bugs WHERE MATCH(summary, description) AGAINST ('crash');
		SELECT * FROM Bugs WHERE MATCH(summary, description) AGAINST ('+crash -save' IN BOOLEAN MODE);
		```
		
	- _Oracle:_ Provides `CONTEXT`, `CTXCAT`, and `CTXXPATH` index types along with operators like `CONTAINS()`, and `CATSEARCH()`.
		
		```sql
		CREATE INDEX BugsText ON Bugs(summary) INDEXTYPE IS CTSSYS.CONTEXT;
		SELECT * FROM Bugs WHERE CONTAINS(summary, 'crash') > 0;
		```
		
	- _Microsoft SQL Server:_ Offers full-text indexing and the `CONTAINS()` operator.
		
		```sql
		EXEC sp_fulltext_database 'enable'
		EXEC sp_fulltext_catalog 'BugsCatalog', 'create'
		EXEC sp_fulltext_table 'Bugs', 'create', 'BugsCatalog', 'bug_id'
		EXEC sp_fulltext_column 'Bugs', 'summary', 'add', '2057'
		EXEC sp_fulltext_table 'Bugs', 'activate'
		SELECT * FROM Bugs WHERE CONTAINS(summary, '"crash"');
		```
		
	- _PostgreSQL:_ Uses the `TSVECTOR` data type, the `@@` operator, and `GIN` indexes for text search.
		
		```sql
		CREATE TABLE Bugs (
		bug_id SERIAL PRIMARY KEY,
		summary VARCHAR(80),
		description TEXT,
		ts_bugtext TSVECTOR
		-- other columns
		);
		CREATE TRIGGER ts_bugtext BEFORE INSERT OR UPDATE ON Bugs
		FOR EACH ROW EXECUTE PROCEDURE
		tsvector_update_trigger(ts_bugtext, 'pg_catalog.english', summary, description);
		CREATE INDEX bugs_ts ON Bugs USING GIN(ts_bugtext);
		SELECT * FROM Bugs WHERE ts_bugtext @@ to_tsquery('crash');
		```
		
	- _SQLite:_ Offers an optional extension FTS3, FTS4.
		
		```sql
		CREATE VIRTUAL TABLE BugsText USING fts3(summary, description);
		INSERT INTO BugsText (docid, summary, description)
		SELECT bug_id, summary, description FROM Bugs;
		SELECT b.* FROM BugsText t JOIN Bugs b ON (t.docid = b.bug_id)
		WHERE BugsText MATCH 'crash';
		```
		
- **Third-Party Search Engines:** Consider external search engines, such as Sphinx Search or Apache Lucene.
	
- **Inverted Index:** Create an inverted index to improve search.
	
	- Create `Keywords` and `BugsKeywords` tables:
		
		```sql
		CREATE TABLE Keywords (
		keyword_id SERIAL PRIMARY KEY,
		keyword VARCHAR(40) NOT NULL,
		UNIQUE KEY (keyword)
		);
		CREATE TABLE BugsKeywords (
		keyword_id BIGINT UNSIGNED NOT NULL,
		bug_id BIGINT UNSIGNED NOT NULL,
		PRIMARY KEY (keyword_id, bug_id),
		FOREIGN KEY (keyword_id) REFERENCES Keywords(keyword_id),
		FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id)
		);
		```
		
	- Stored procedure example:
		
		```sql
	CREATE PROCEDURE BugsSearch(keyword VARCHAR(40))
	BEGIN
	SET @keyword = keyword;
	PREPARE s1 FROM 'SELECT MAX(keyword_id) INTO @k FROM Keywords
	WHERE keyword = ?';
	EXECUTE s1 USING @keyword;
	DEALLOCATE PREPARE s1;
	IF (@k IS NULL) THEN
		PREPARE s2 FROM 'INSERT INTO Keywords (keyword) VALUES (?)';
		EXECUTE s2 USING @keyword;
		DEALLOCATE PREPARE s2;
		SELECT LAST_INSERT_ID() INTO @k;
		PREPARE s3 FROM 'INSERT INTO BugsKeywords (bug_id, keyword_id)
		SELECT bug_id, ? FROM Bugs
		WHERE summary REGEXP CONCAT('\'[[:<:]]\'', ?, '\'[[:>:]]\'')
		OR description REGEXP CONCAT('\'[[:<:]]\'', ?, '\'[[:>:]]\'');
		EXECUTE s3 USING @k, @keyword, @keyword;
		DEALLOCATE PREPARE s3;
	END IF;
	PREPARE s4 FROM 'SELECT b.* FROM Bugs b
	JOIN BugsKeywords k USING (bug_id)
	WHERE k.keyword_id = ?';
	EXECUTE s4 USING @k;
	DEALLOCATE PREPARE s4;
	END
		```

![[Pasted image 20251225162056.png]]
➊ Search for the user-specified keyword. Return either the integer primary key from Keywords.keyword_id or null if the word has not been seen previously. 
➋ If the word was not found, insert it as a new word. 
➌ Query for the primary key value generated in Keywords. 
➍ Populate the intersection table by searching Bugs for rows containing the new keyword. 
➎ Finally, query the full rows from Bugs that match the keyword_id, whether the keyword was found or had to be inserted as a new entry.
# Notes

## Sphinx Search

**Sphinx Search** is an open-source full-text search engine designed to provide fast and relevant text search capabilities. Here's what it can do:

**Key Features:**

- **High-performance full-text search** with advanced ranking algorithms
- **Boolean, phrase, and proximity searches** with support for complex queries
- **Real-time indexing** capabilities for frequently updated data
- **Faceted search** and filtering
- **Scalability** for handling millions of documents
- **Multiple data source support** including MySQL, PostgreSQL, and XML
- **Relevance ranking** with customizable weighting

### Using Sphinx with MySQL

**Basic Integration Steps:**

1. **Install Sphinx** on your server alongside MySQL
    
2. **Configure Sphinx** (sphinx.conf file):
    
    - Define data sources pointing to your MySQL database
    - Specify which tables and columns to index
    - Set up indexing parameters
3. **Create indexes** from your MySQL data:
    
    ```bash
    indexer --all
    ```
    
4. **Start the Sphinx daemon**:
    
    ```bash
    searchd
    ```
    
5. **Query from your application**:
    
    - Use SphinxAPI or SphinxQL (MySQL protocol)
    - SphinxQL allows you to query Sphinx using familiar SQL-like syntax

**Example SphinxQL Query:**

```sql
SELECT * FROM index_name WHERE MATCH('search terms') LIMIT 10;
```

Sphinx acts as a separate search layer that indexes your MySQL data and provides much faster full-text search compared to MySQL's native FULLTEXT indexes, especially for large datasets.