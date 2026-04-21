# PES Version Control System (pes-vcs)

## Overview

This project implements a simplified Git-like version control system called **PES**. It supports object storage, tree structures, indexing (staging), and commits. The system mimics Git internals such as content-addressable storage, tree snapshots, and commit history.

---

## Phase 1: Object Storage

### Screenshot 1A: Test Objects Output

<img width="720" height="128" alt="{9364F909-F02E-48F2-8354-D56F349ABCEB}" src="https://github.com/user-attachments/assets/42cd250c-2fdc-4c7c-a45d-4ffc2e129186" />



### Screenshot 1B: Object Storage Structure

Command used:

```bash
find .pes/objects -type f
```

<img width="719" height="73" alt="{58B7A607-4380-4955-82AE-CB63739ABF1D}" src="https://github.com/user-attachments/assets/be8d601c-3483-4461-9825-c70646c1da7a" />


---

## Phase 2: Tree Objects

### Screenshot 2A: Test Tree Output

<img width="719" height="141" alt="{CAD4D9BF-EF8D-4BCE-BE08-C3512C0919B8}" src="https://github.com/user-attachments/assets/bc61fdf3-4501-4e8e-b57b-8fd34e3ea9fd" />


### Screenshot 2B: Raw Tree Object

Command used:

```bash
xxd .pes/objects/<XX>/<YYY...> | head -20
```

<img width="611" height="418" alt="{AEB5E0F4-7FF2-4520-9373-505762A81E2A}" src="https://github.com/user-attachments/assets/1d963bbc-46bd-450a-a1e4-76714b72f949" />


---

## Phase 3: Index (Staging Area)

### Screenshot 3A: Init → Add → Status

Commands used:

```bash
./pes init
./pes add file1.txt file2.txt
./pes status
```
<img width="610" height="107" alt="{19177D35-51DA-4830-8347-8E98BA3F02DA}" src="https://github.com/user-attachments/assets/9b7bb869-2873-48e3-9a0c-b3ad27aea96b" />

<img width="452" height="344" alt="{4944D956-E0F2-427E-AAF7-1E607FB92352}" src="https://github.com/user-attachments/assets/21838984-9f53-4389-8acd-cce83ad0369c" />

### Screenshot 3B: Index File

Command used:

```bash
cat .pes/index
```

<img width="452" height="41" alt="{83C530BA-6A36-41A4-9192-882FA5933D7E}" src="https://github.com/user-attachments/assets/82ae03f4-5cd0-4b52-af5a-0509baf9b970" />


---

## Phase 4: Commits

### Screenshot 4A: Commit Log

Command used:

```bash
./pes log
```

<img width="363" height="357" alt="{32370047-6F32-4A8E-9AA7-95E2E18EABF5}" src="https://github.com/user-attachments/assets/d34c5ac0-08b1-45f1-977b-8a3053532049" />


### Screenshot 4B: Object Growth

Command used:

```bash
find .pes -type f | sort
```

<img width="361" height="381" alt="{9E174B59-B74D-4022-98CE-5591EEBFFD9A}" src="https://github.com/user-attachments/assets/00d90a5b-441d-4df7-ae8b-9f14e7dd4a0c" />

<img width="352" height="386" alt="{16EFB783-0A72-48E7-995A-10F3A9C6784D}" src="https://github.com/user-attachments/assets/f88baa16-dc60-4db8-a996-031799e21977" />

### Screenshot 4C: HEAD and Branch Reference

Commands used:

```bash
cat .pes/refs/heads/main
cat .pes/HEAD
```

<img width="363" height="52" alt="{C0953323-DD47-4A86-8FC3-C1B8B41A9F39}" src="https://github.com/user-attachments/assets/4dd5d83d-c8ad-4028-a30e-6d920fb28783" />


---

# Conceptual Questions

## Q5.1

A branch in Git is a file storing a commit hash, so `pes checkout <branch>` would involve updating `.pes/HEAD` to point to the selected branch and reading the commit hash from `.pes/refs/heads/<branch>`. The system must then reconstruct the working directory using the tree associated with that commit. The index should also be updated to match this snapshot. The complexity arises in safely modifying files without overwriting uncommitted changes. Handling nested directories and file differences adds to the difficulty.

---

## Q5.2

To detect a dirty working directory, compare each file in the index with the current filesystem using metadata such as size and modification time. If a file has changed and also differs between the current branch and the target branch, it creates a conflict. A stronger method is to recompute the hash and compare it with the stored blob hash. If mismatches exist, checkout must be aborted to prevent overwriting changes. This ensures data integrity during branch switching.

---

## Q5.3

In a detached HEAD state, commits are created but not referenced by any branch. These commits become unreachable unless explicitly saved. Users can recover them by using the commit hash and creating a new branch pointing to it. Until garbage collection runs, these commits remain in the object store. Detached HEAD is useful for temporary work but risky if not preserved.

---

## Q6.1

Garbage collection identifies unreachable objects by starting from all branch heads and traversing commits, trees, and blobs recursively. A hash set can be used to track visited objects efficiently. After traversal, any object not in the reachable set can be safely deleted. For a repository with 100,000 commits and 50 branches, the traversal would visit all reachable commits and their associated trees and blobs, potentially hundreds of thousands of objects. This process ensures storage efficiency.

---

## Q6.2

Running garbage collection concurrently with commits is dangerous because GC might delete objects that are not yet referenced but are in the process of being written. For example, a commit writes objects but has not updated HEAD, making them appear unreachable. GC could delete these objects, breaking the commit. Git avoids this using atomic operations, temporary files, and delayed cleanup. Locking mechanisms and grace periods ensure safe garbage collection.

---

## Conclusion

This project demonstrates core version control concepts including object storage, tree structures, indexing, commits, and repository management. It provides insight into how systems like Git internally manage data efficiently and reliably.
