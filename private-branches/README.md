# Selective Branch Publishing & Public Fork Control

**Author:** [Gokul Gireesh](https://github.com/Gokul-Gireesh)  
**Category:** Version Control / Infrastructure Efficiency  
**Status:** Feature Request  

## 1. The Crisis: Fork Bloat & Resource Waste
Currently, GitHub forks are "all-or-nothing." This creates a massive technical and resource crisis for large-scale projects:

*   **Data Overhead:** When a user forks a repository to contribute a 1KB fix, they are often forced to clone gigabytes of internal staging branches, experimental features, and historical data they don't need.
*   **Infrastructure Strain:** GitHub’s servers must process and transmit the entire packfile, including "hidden" history from branches that the public will never see.
*   **Developer Friction:** For developers in regions with limited bandwidth, forking a large project with hundreds of internal branches becomes impossible, even if they only want to work on the `main` branch.

## 2. The Proposal: Selective Branch Publishing
This feature allows repository owners to designate specific branches as **Public** or **Private** within a single repository. 

*   **Public Branches:** The only refs advertised to the world.
*   **Private Branches:** Stay local to authorized collaborators.
*   **Clean Forks:** A public fork only contains the history **reachable** from public branches. Anything else is literally stripped out of the fork's storage.

## 3. Technical Implementation
By leveraging **Ref Filtering** and **Reachability Analysis**, GitHub can ensure that private "dangling" commits are never included in the public packfile:

*   **Filtered Ref Advertisement:** Public `git clone` or `git fetch` calls only see public `refs/heads/`.
*   **Optimized Packfiles:** During a fork/clone, the server excludes objects that are only reachable from private branches. 
*   **Efficiency:** This reduces the disk footprint of millions of forks across the platform.

## 4. Key Benefits
*   **Massive Storage Savings:** Forks become lightweight, containing only the code necessary for public contribution.
*   **Security by Design:** Prevents the accidental exposure of sensitive internal architecture or roadmap-leaking commits.
*   **Reduced Bandwidth:** Millions of developers save time and data by cloning only the active public "surface" of a project.
*   **No Repo Fragmentation:** Eliminate the need for "Mirror Repos" (one private for dev, one public for community). Everything stays under one roof, but behind a smart visibility layer.

## 5. Configuration Example
In the Repository Settings:
*   `main` (Public) -> **Size: 50MB**
*   `v1-stable` (Public) -> **Size: 45MB**
*   `internal-dev-v2` (Private) -> **Size: 2GB (Hidden from forks)**

---

## 🤝 Community Feedback
This isn't just a feature; it's a way to make Git more authoritative and the GitHub platform more sustainable.

**Are you interested in this?** Please open an issue in the root of this [proposals repo](https://github.com/Gokul-Gireesh/proposal) to discuss.