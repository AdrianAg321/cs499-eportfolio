# CS 340 Artifact – CRUD Python Module (Original and Enhanced)

This artifact comes from the CS 340 Client/Server Development course. The assignment required building a CRUD module in Python that connects to MongoDB and supports Create, Read, Update, and Delete operations for the Grazioso Salvare AAC dataset. The module served as the backend for the dashboard application used in Project Two.

I selected this artifact because it demonstrates my development in database operations, error handling, secure configuration, and the organization of backend logic. The enhanced version improves structure, introduces logging, validates inputs, and adds professional coding practices that strengthen reliability and maintainability.

---

# Original Code

```python
"""
CRUD_Python_Module.py
Used by Project Two dashboard to interface with MongoDB.
Contains CRUD operations for the Grazioso Salvare AAC dataset.
"""

from typing import Dict, List
from pymongo import MongoClient
from pymongo.errors import PyMongoError

class AnimalShelter(object):
    """CRUD operations for Animal collection in MongoDB"""

    def __init__(
        self,
        username: str = "aacuser",
        password: str = "SNHU",
        host: str = "localhost",
        port: int = 27017,
        db: str = "aac",
        collection: str = "animals",
    ):
        # Basic connection string used throughout CS 340
        uri = f"mongodb://{username}:{password}@{host}:{port}/{db}?authSource=admin"
        self.client = MongoClient(uri, serverSelectionTimeoutMS=4000)

        # Early connection check
        self.client.admin.command("ping")

        self.database = self.client[db]
        self.collection = self.database[collection]

    def create(self, data: Dict) -> bool:
        try:
            if not isinstance(data, dict) or not data:
                return False
            result = self.collection.insert_one(data)
            return bool(result.acknowledged and result.inserted_id)
        except PyMongoError:
            return False

    def read(self, query: Dict) -> List[Dict]:
        try:
            if not isinstance(query, dict):
                return []
            return list(self.collection.find(query))
        except PyMongoError:
            return []

    def update(self, query: Dict, update_doc: Dict, many: bool = False) -> int:
        try:
            if many:
                result = self.collection.update_many(query, update_doc)
            else:
                result = self.collection.update_one(query, update_doc)
            return int(result.modified_count)
        except PyMongoError:
            return 0

    def delete(self, query: Dict, many: bool = False) -> int:
        try:
            if many:
                result = self.collection.delete_many(query)
            else:
                result = self.collection.delete_one(query)
            return int(result.deleted_count)
        except PyMongoError:
            return 0
```

---

# Enhanced Code

```python
from typing import Dict, List, Optional
import os
import logging

from pymongo import MongoClient
from pymongo.collection import Collection
from pymongo.errors import PyMongoError, ServerSelectionTimeoutError

# My Note:
# In this enhanced version, I focused on improving structure, security, and maintainability.
# The original CS340 module worked, but it relied on hard-coded credentials,
# lacked validation, and returned silent failures. This enhanced version
# introduces professional patterns I learned later in the program.

# ----------------------------------------------------------------------
# Logging configuration
# ----------------------------------------------------------------------
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s - %(message)s",
)
logger = logging.getLogger(__name__)


class AnimalShelter(object):
    """CRUD operations for the 'animals' collection in MongoDB."""

    # Constructor
    def __init__(
        self,
        username: Optional[str] = None,
        password: Optional[str] = None,
        host: Optional[str] = None,
        port: Optional[int] = None,
        db: str = "aac",
        collection: str = "animals",
        timeout_ms: int = 4000,
    ) -> None:
        """
        Initializes the AnimalShelter with safer and more flexible configuration handling.
        Supports environment variables to avoid hard-coded credentials.
        """

        # My Note:
        # This improves security by removing hard-coded usernames/passwords.
        # It also makes the module easier to use in different environments.
        self.username = username or os.getenv("AAC_DB_USERNAME", "aacuser")
        self.password = password or os.getenv("AAC_DB_PASSWORD", "SNHU")
        self.host = host or os.getenv("AAC_DB_HOST", "localhost")
        self.port = port or int(os.getenv("AAC_DB_PORT", "27017"))

        uri = (
            f"mongodb://{self.username}:{self.password}"
            f"@{self.host}:{self.port}/{db}?authSource=admin"
        )

        try:
            self.client: MongoClient = MongoClient(
                uri, serverSelectionTimeoutMS=timeout_ms
            )
            self.client.admin.command("ping")

            logger.info("Successfully connected to MongoDB at %s:%s", self.host, self.port)

        except ServerSelectionTimeoutError as ex:
            logger.error("MongoDB connection timed out: %s", ex)
            raise
        except PyMongoError as ex:
            logger.error("MongoDB connection error: %s", ex)
            raise

        self.database = self.client[db]
        self.collection: Collection = self.database[collection]

    # Internal validation helpers
    def _validate_document(self, data: Dict) -> bool:
        if not isinstance(data, dict):
            logger.warning("Validation failed: data is not a dict.")
            return False
        if not data:
            logger.warning("Validation failed: data is empty.")
            return False
        return True

    def _validate_query(self, query: Dict) -> bool:
        if not isinstance(query, dict):
            logger.warning("Validation failed: query is not a dict.")
            return False
        return True

    # Create
    def create(self, data: Dict) -> bool:
        if not self._validate_document(data):
            return False

        try:
            result = self.collection.insert_one(data)
            success = bool(result.acknowledged and result.inserted_id)
            if success:
                logger.info("Inserted document with _id=%s", result.inserted_id)
            return success
        except PyMongoError as ex:
            logger.error("Create operation failed: %s", ex)
            return False

    # Read
    def read(self, query: Dict, limit: int = 0, skip: int = 0) -> List[Dict]:
        if not self._validate_query(query):
            return []

        try:
            cursor = self.collection.find(query)

            # My Note:
            # Adding pagination support makes the module more flexible
            # and prevents overwhelming the caller with thousands of records.
            if skip > 0:
                cursor = cursor.skip(skip)
            if limit > 0:
                cursor = cursor.limit(limit)

            results = list(cursor)
            logger.info("Read operation returned %d documents.", len(results))
            return results
        except PyMongoError as ex:
            logger.error("Read operation failed: %s", ex)
            return []

    # Update
    def update(self, query: Dict, update_doc: Dict, many: bool = False) -> int:
        if not self._validate_query(query) or not isinstance(update_doc, dict):
            logger.warning("Invalid update request.")
            return 0

        try:
            result = self.collection.update_many(query, update_doc) if many else \
                     self.collection.update_one(query, update_doc)

            modified = int(result.modified_count)
            logger.info("Updated %d documents.", modified)
            return modified
        except PyMongoError as ex:
            logger.error("Update operation failed: %s", ex)
            return 0

    # Delete
    def delete(self, query: Dict, many: bool = False) -> int:
        if not self._validate_query(query):
            logger.warning("Invalid delete request.")
            return 0

        try:
            result = self.collection.delete_many(query) if many else \
                     self.collection.delete_one(query)

            deleted = int(result.deleted_count)
            logger.info("Deleted %d documents.", deleted)
            return deleted
        except PyMongoError as ex:
            logger.error("Delete operation failed: %s", ex)
            return 0
```

---

# Summary of Enhancements

The enhanced version improves:

- **Security** by using environment variables  
- **Reliability** via validation helpers  
- **Maintainability** with structured logging  
- **Error visibility** using try/except blocks with logging  
- **Scalability** with paging support in the read operation  

These changes reflect the skills I developed later in the program and help make the module more professional and production-ready.

---

