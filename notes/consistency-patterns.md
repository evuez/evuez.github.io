title = "Consistency patterns"
%%%

- **Eventual consistency**: Eventually, all reads will return the last updated value
- **Monotonic reads**: A read will always return the same value or a more recent value than the previous read
- **Monotonic writes**: For a given process, any write is completed before any successive write operation
- **Read-your-writes**: Any write by a given process will be available to that same process for read operations
- **Sequential consistency**: Writes aren't necessarily seen instantaneously, but ordering of writes appears consistent across all processes
- **Strict consistency**: Every parallel process observes the same consistent state
