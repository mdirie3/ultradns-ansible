# UltraDNS Quiz Application Log Analysis

## Overview

This project uses Ansible to automate the analysis of UltraDNS quiz application timing logs across multiple servers.

The playbook retrieves the latest 20 matching log entries and calculates application response-time statistics for each server.

The playbook:
- Checks whether the application log exists
- Retrieves the latest 20 matching timing log entries
- Extracts application response times in milliseconds
- Calculates the mean response time
- Calculates the standard deviation
- Displays the analysis results for each server
- Handles cases where no matching log entries are found

## Requirements

- Ansible
- Linux or WSL environment for local testing
- SSH access to the target servers
- Python installed on the target servers
- Access to the application log at `/var/log/ultradns-quiz-madeup.log`

## Project Structure

```text
.
├── playbook.yml
├── inventory.ini
├── test_inventory.ini
├── madeup.log
└── screenshots/

```

## Files

- `playbook.yml` - Ansible playbook used to analyze the application timing logs.
- `inventory.ini` - Inventory containing the target quiz application servers.
- `test_inventory.ini` - Local inventory used to test the playbook without requiring access to the target servers.
- `madeup.log` - Sample log data used for local testing.
- `screenshots/` - Screenshots demonstrating the successful local test and project structure.

## Inventory

### Target Server Inventory

The `inventory.ini` file contains the target quiz application servers:

```ini
[quiz_servers]
server00.quiz.example.vercara
server01.quiz.example.vercara
server02.quiz.example.vercara
server03.quiz.example.vercara
server04.quiz.example.vercara
server05.quiz.example.vercara
```

### Local Test Inventory

The `test_inventory.ini` file is used to test the playbook locally:

```ini
[quiz_servers]
localhost ansible_connection=local
```

This allows the playbook to be validated locally before running it against the target servers.

## Log File

The playbook analyzes the application log located at:

```text
/var/log/ultradns-quiz-madeup.log
```

For local testing, the sample `madeup.log` file contains timing entries matching the following format:

```text
Ultradns quiz application took <number> ms
```

The playbook uses a regular expression to identify the relevant timing entries and extract the response time in milliseconds.

## Playbook Workflow

The playbook performs the following steps:

1. Connects to the servers defined in the `quiz_servers` inventory group.
2. Searches the application log for matching UltraDNS timing entries.
3. Retrieves the latest 20 matching entries.
4. Extracts the timing values from the log entries.
5. Calculates the mean response time.
6. Calculates the sum of squared differences from the mean.
7. Calculates the standard deviation.
8. Displays the server name, timing values, number of samples, mean, and standard deviation.
9. Reports when no matching timing entries are found.

## Running the Playbook

### Local Testing

To test the playbook locally using the test inventory:

```bash
ansible-playbook -i test_inventory.ini playbook.yml
```

### Target Servers

To run the playbook against the target servers:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

The target servers must be reachable through Ansible and must have the expected application log available at:

```text
/var/log/ultradns-quiz-madeup.log
```

## Example Output

A successful local test produced the following results:

```text
Server: localhost
Timings (ms): [9, 12, 7, 7, 9, 10, 11, 13, 11, 7, 7, 8, 12, 14, 11, 10, 10, 11, 9, 10]
Samples: 20
Mean: 9.9 ms
Std Dev: 2.0 ms
```

The playbook completed successfully with:

```text
ok=7
changed=0
unreachable=0
failed=0
skipped=2
```

The skipped task is expected when timing entries are found because the playbook only reports "no matching entries found" when zero timing entries are available.

## Design Decisions

### `ansible.builtin.stat`

The `stat` module checks whether the expected log file exists before attempting to process it.

This allows the playbook to handle a missing log file separately from a log file that exists but contains no matching timing entries.

### `gather_facts: false`

System facts are not required for this log-analysis task, so fact gathering is disabled to avoid unnecessary work.

### `ansible.builtin.shell`

The shell module is used to search the log with `grep` and retrieve the latest 20 matching entries with `tail`.

The command is read-only and does not modify the target server. The task is also marked with `changed_when: false` because the playbook is only reading log data.

The operation is read-only and does not modify the target server.

### `changed_when: false`

The log analysis does not make any changes to the target system, so the shell task is explicitly marked as unchanged.

### `failed_when`

The playbook allows `grep` return codes `0` and `1`:

- `0` - Matching entries were found.
- `1` - No matching entries were found.
- Other return codes - Treated as an error.

This prevents the playbook from incorrectly treating a normal "no matches" result as a failure.

### `ansible.builtin.set_fact`

`set_fact` is used to store values calculated during the playbook execution, including:

- Extracted timing values
- Mean response time
- Sum of squared differences
- Standard deviation

These values can then be used by subsequent tasks.

### `ansible.builtin.debug`

The debug module is used to display the final analysis results, including the timing values, sample count, mean, and standard deviation.

### Handling Empty Results

The calculation tasks only run when timing values are available:

```text
timings | length > 0
```

If no matching entries are found, the playbook instead reports:

```text
No matching timing entries found
```

This prevents calculations from being performed against an empty dataset.

## Screenshots

Screenshots demonstrating the successful local test and project structure and used madeup log are in the screenshot folder.
