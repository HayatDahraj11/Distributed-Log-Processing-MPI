# Parallel Log File Analysis Using MPI

**Hayat Sikandar Dahraj**  
*October 7, 2024*

## Abstract

This report presents the development and implementation of a parallel log file analysis system using MPI (Message Passing Interface) to detect signs of malicious activity in large-scale system log files. Given the massive size of log files generated by modern systems, a parallel approach is essential for efficient data processing. The UNSW-NB15 dataset was utilized, encompassing nine types of network attacks, to simulate real-world scenarios. The system leverages MPI's collective communication functions to distribute workload, aggregate results, and detect distributed attacks across multiple processes.

## Contents

1. [Introduction](#introduction)
2. [Problem Description](#problem-description)
3. [Dataset Description](#dataset-description)
4. [Implementation](#implementation)
   - 4.1. [Log File Distribution](#log-file-distribution)
   - 4.2. [Pattern Matching](#pattern-matching)
   - 4.3. [Inter-Process Communication](#inter-process-communication)
   - 4.4. [Checksum Validation](#checksum-validation)
   - 4.5. [Performance Analysis](#performance-analysis)
5. [Results](#results)
   - 5.1. [Suspicious Activity Detection](#suspicious-activity-detection)
   - 5.2. [Checksum Validation](#checksum-validation-results)
   - 5.3. [Suspicious IP Addresses](#suspicious-ip-addresses)
   - 5.4. [Performance Analysis](#performance-analysis-results)
6. [Discussion](#discussion)
   - 6.1. [Communication Overhead](#communication-overhead)
   - 6.2. [Scalability](#scalability)
7. [Conclusion](#conclusion)
8. [Future Work](#future-work)
9. [Appendix](#appendix)

## Introduction

As networked systems become increasingly complex, the volume of generated log data grows exponentially. Analyzing these logs for signs of malicious activity is critical for maintaining system security but poses significant computational challenges due to the data size. Traditional serial log analysis methods are insufficient for timely detection of threats.

This project aims to develop a parallel log file analysis system using MPI to efficiently process large log files and detect various types of network attacks. By distributing the workload across multiple processes and utilizing MPI's communication capabilities, the system can analyze logs faster and detect distributed attacks that may not be apparent in isolated data segments.

## Problem Description

### Objectives:
- **Efficient Log Processing**: Develop a parallel solution to process large log files using MPI.
- **Pattern Detection**: Implement pattern matching to identify suspicious activities indicative of malicious behavior.
- **Distributed Attack Detection**: Aggregate results from all processes to detect coordinated attacks across multiple IP addresses.
- **Validation and Error Checking**: Ensure the integrity and correctness of the analysis through checksum validation.
- **Performance Evaluation**: Measure and compare the performance of the parallel implementation against a serial approach.

## Dataset Description

The UNSW-NB15 dataset was used for this project. It contains 2,540,044 records divided across four CSV files:

- UNSW-NB15_1.csv
- UNSW-NB15_2.csv
- UNSW-NB15_3.csv
- UNSW-NB15_4.csv

Each record represents a network flow with features such as source IP, destination IP, protocol, and attack label. The dataset includes nine types of attacks:

1. Fuzzers
2. Analysis
3. Backdoors
4. DoS (Denial of Service)
5. Exploits
6. Generic
7. Reconnaissance
8. Shellcode
9. Worms

This diverse range of attacks makes the dataset suitable for testing the effectiveness of the log analysis system in detecting various malicious activities.

## Implementation

### Log File Distribution
The master process reads and combines all four CSV files into a single data buffer. To distribute the workload:

1. The combined log data is divided into chunks, ensuring that no log entries are split between chunks by aligning on newline characters.
2. MPI_Scatterv is used to distribute variable-sized chunks to each process.
3. Each process receives its chunk and proceeds to analyze it independently.

### Pattern Matching
Each process scans its assigned chunk for suspicious patterns using regular expressions. The patterns correspond to the nine attack types in the dataset. The regex.h library is used for compiling and executing the regular expressions.

Steps:
1. Read and temporarily store each log entry.
2. Apply regular expressions to detect any of the attack patterns.
3. If a match is found:
   - Increment the local suspicious activity count.
   - Increment the specific pattern count.
   - For attacks such as Backdoors, DoS, and Reconnaissance, extract and store the source IP address for further analysis.

### Inter-Process Communication
After local analysis, the processes collaborate to aggregate results using MPI:

- **MPI_Reduce**: Aggregates suspicious activity counts and pattern-specific counts across all processes, sending the results to the master process.
- **MPI_Gather**: Collects lists of suspicious IPs from all processes.
- **Master Process Aggregation**:
  - Cross-checks and removes duplicate IPs.
  - Creates a final list of unique suspicious IPs.
- **MPI_Bcast**: Broadcasts the final list of suspicious IPs to all processes.
- **MPI_Allreduce**: Counts global occurrences of suspicious IPs across all processes.

### Checksum Validation
Each process calculates a local checksum for its chunk by summing the ASCII values of all characters. These are aggregated using MPI_Reduce to produce a global checksum, validating the integrity of the data processing.

### Performance Analysis
The performance is measured using MPI_Wtime:
- Timing starts after data distribution and ends after analysis and communication.
- The serial analysis is also timed for comparison.
- Tests with varying process counts assess scalability.

## Results

### Suspicious Activity Detection
Total Suspicious Activities Detected: 321,283

Breakdown by Attack Type:

| Attack Type     | Count   |
|----------------|---------|
| Fuzzers        | 24,246  |
| Analysis       | 2,677   |
| Backdoor       | 2,329   |
| DoS            | 16,353  |
| Exploit        | 44,525  |
| Generic        | 215,481 |
| Reconnaissance | 13,987  |
| Shellcode      | 1,511   |
| Worms          | 174     |

### Checksum Validation Results
Global Checksum: 29,833,234,424

This confirms all data segments were processed without omissions or duplication.

### Suspicious IP Addresses

| IP Address      | Occurrences |
|----------------|-------------|
| 149.171.126.10 | 44,059     |
| 149.171.126.11 | 14,271     |
| 149.171.126.12 | 30,484     |

### Performance Analysis Results

| Analysis Type | Processes | Execution Time (s) |
|--------------|-----------|-------------------|
| Serial       | 1         | 13.11            |
| Parallel     | 2         | 6.89             |
| Parallel     | 4         | 3.99             |

## Discussion

### Communication Overhead
MPI communication introduces synchronization overhead, especially for operations like MPI_Gather and MPI_Allreduce, but the performance gains outweigh the cost.

### Scalability
The system demonstrates good scalability with increasing processes, but performance gains diminish as communication overhead increases.

## Conclusion

The project successfully implemented a scalable parallel log file analysis system using MPI, achieving significant speedup and efficiency in detecting malicious activities.

## Future Work

- Dynamic Load Balancing
- Integration of Machine Learning for anomaly detection
- Real-time log processing

## Appendix

### Source Code
Available on request.

### Compilation and Execution Instructions

**Compilation:**
```bash
mpicc -o mpi_log_analysis mpi_log_analysis.c -lm
```

**Execution:**
```bash
# For 2 processes:
mpirun -np 2 ./mpi_log_analysis

# For 4 processes:
mpirun -np 4 ./mpi_log_analysis
```
