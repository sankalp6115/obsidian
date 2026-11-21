Project Roadmap: Tiny Persistent Key-Value Store
Project start
 7 checkpoints

Phase 1 — Core functionality
1. Checkpoint 1: Basic in-memory key-value store
   
   Build the simplest possible key-value store using a Python dictionary.
   
   Tasks:
   - Implement put(key, value).
   - Implement get(key).
   - Implement delete(key).
   - Handle nonexistent keys.

   Expected result:
- store.put("name", "Sankalp")
- store.get("name")       # Sankalp
- store.delete("name")
- store.get("name")       # None
   
   
   
Completion criteria: All three operations work correctly without using disk storage.
   
2. Checkpoint 2: Introduce append-only persistence
   
   Instead of relying only on a dictionary, persist every operation in store.log.
   
   Tasks:
   - Create store.log automatically if it doesn't exist.
   - Every put appends a PUT record.
   - Every delete appends a DELETE record.
   - Keep the dictionary updated alongside the log.
   Expected log:
   PUT name Sankalp
   PUT language Python
   DELETE language
   
   
   Completion criteria
   Every operation is recorded on disk. You can inspect the log and see the history of operations.
3. Checkpoint 3: Recover state from the log
   
   Currently, your dictionary disappears when the program exits. Fix that.
   
   Tasks:
   - Read store.log during initialization.
   - Replay each operation in chronological order.
   - Reconstruct the in-memory dictionary.
   - Ensure updates overwrite previous values.
   - Ensure deletes remove previously stored keys.
   Expected result:
   
   Given:
   PUT name Sankalp
   PUT language Python
   PUT name Sankalp2
   DELETE language
   
   
   Recovered dictionary:
   {"name": "Sankalp2"}
   
   
   
   Completion criteria
   Stop and restart the application. Previously stored data must still be available.

Phase 2 — Usability and reliability
5. Checkpoint 4: Build the CLI interface
   
   Make the storage engine usable from the terminal.
   
   Support:
   python store.py put name Sankalp
   python store.py get name
   python store.py delete name
   
   
   Tasks:
   - Parse command-line arguments.
   - Connect CLI commands to your storage engine.
   - Display appropriate success/error messages.
   - Handle invalid commands and missing arguments.
   Completion criteria
   All operations work through the terminal, including across separate executions of the program.

5. Checkpoint 5: Handle malformed and corrupted records
   
   Your storage engine shouldn't crash because of one bad log entry.
   
   Tasks:
   - Detect malformed records.
   - Skip invalid records during recovery.
   - Handle unknown operation types.
   - Handle missing keys or values.
   - Report malformed entries with their line numbers.
   Completion criteria
   A malformed record doesn't prevent valid subsequent records from being recovered.


5. Checkpoint 6: Introduce JSON Lines and crash recovery
   
   Improve your log format.
   
   Replace space-separated records with JSON Lines (JSONL).
   
   Example:
   {"op":"PUT","key":"name","value":"Sankalp"}
   {"op":"DELETE","key":"language"}
   
   
   Tasks:
   - Serialize each operation as JSON.
   - Write one JSON object per line.
   - Handle incomplete final records.
   - Ensure a partially written final record doesn't corrupt previously committed data.
   - Consider flushing writes and using fsync() for durability.
   Completion criteria
   The application can recover from a log containing valid records followed by an incomplete final JSON record.
Phase 3 — Storage optimization
1. Checkpoint 7: Implement log compaction
   
   Your log grows indefinitely because every operation is preserved.
   
   Build a mechanism to rewrite the log using only the latest state.
   
   Example:
   
   Before compaction:
   PUT name Sankalp
   PUT language Python
   PUT name Sankalp2
   DELETE language
   PUT city Kanpur
   
   
   After compaction:
   {"op":"PUT","key":"name","value":"Sankalp2"}
   {"op":"PUT","key":"city","value":"Kanpur"}
   
   
   Tasks:
   - Generate records from the current dictionary.
   - Write them into a temporary file.
   - Safely replace the old log.
   - Ensure the application can recover after compaction.
   - Preserve the original log if compaction fails before replacement.
   Completion criteria
   Compaction reduces log size without changing the current key-value state.

Final project structure
You don't need to create all these files at the beginning. This is a possible final structure:
tiny-kv-store/
│
├── store.py
├── storage.py
├── store.log
├── tests/
│   ├── test_basic.py
│   ├── test_recovery.py
│   ├── test_corruption.py
│   └── test_compaction.py
│
└── README.md


Final completion checklist
Project progress
0%



0 of 7 checkpoints completed
[ ] Checkpoint 1: In-memory CRUD operations
[ ] Checkpoint 2: Append-only log persistence
[ ] Checkpoint 3: Startup log replay and recovery
[ ] Checkpoint 4: Working CLI interface
[ ] Checkpoint 5: Malformed record handling
[ ] Checkpoint 6: JSONL format and partial-write recovery
[ ] Checkpoint 7: Safe log compaction

My recommendation
Follow the checkpoints strictly. Don't jump to JSONL, compaction, or crash recovery before the basic implementation works.

One additional thing: write tests at every checkpoint, rather than waiting until the end.

This project is particularly useful because you'll encounter several fundamental database engineering concepts without hiding the implementation behind SQLite.

Your first milestone is simple: build a dictionary-backed key-value store with three operations. Nothing more.