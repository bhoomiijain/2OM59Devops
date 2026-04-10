# 🤖 COPILOT INSTRUCTIONS — Update DevOps Notes Repo
# Paste this entire file to GitHub Copilot Chat, then attach the markdown files.
# Repo: https://github.com/bhoomiijain/2OM59Devops.git

# ============================================================
# WHAT TO DO (give this to Copilot)
# ============================================================
#
# "I need you to update my GitHub repo at:
#  https://github.com/bhoomiijain/2OM59Devops.git
#
#  Please do the following:
#
#  1. ADD these NEW files (they don't exist yet):
#     → Unit1/09_Docker_Object_Types_and_Filesystem.md
#     → Unit1/10_Container_Interaction_with_Host.md
#     → Unit2/06_GHCR_and_Private_Registries.md
#
#  2. REPLACE (overwrite) these existing files with updated versions:
#     → Unit2/01_Dockerfile_Image_Creation.md
#        (replaces old 01_Dockerfile_Basics.md — rename + replace)
#     → Unit2/02_Docker_Networking.md
#     → Unit2/03_Docker_Storage.md
#
#  3. UPDATE README.md — add the new files to the Unit I and Unit II
#     navigation tables (see updated README section below).
#
#  4. Commit message:
#     'feat: Add new Unit 1 & 2 notes — Object Types, GHCR, Container Interaction, updated Networking/Storage/Dockerfile'
#
#  5. Push to main branch."

# ============================================================
# UPDATED README SECTION — Unit I table (replace existing)
# ============================================================
#
# ## 🗂️ Unit I — Topics Covered
#
# - [1. Basics of DevOps & Why Docker](./Unit1/01_Basics_of_DevOps.md)
# - [2. Virtualization vs Containerization](./Unit1/02_Virtualization_vs_Containerization.md)
# - [3. Container Runtime & Namespaces](./Unit1/03_Container_Runtime_and_Namespaces.md)
# - [4. Control Groups (cgroups)](./Unit1/04_cgroups.md)
# - [5. Container Images & Layers](./Unit1/05_Container_Images_and_Layers.md)
# - [6. Image Registries & Distribution](./Unit1/06_Image_Registries.md)
# - [7. Docker Architecture & Lifecycle](./Unit1/07_Docker_Architecture.md)
# - [8. Basic Docker Commands](./Unit1/08_Basic_Docker_Commands.md)
# - [9. Docker Object Types & Filesystem](./Unit1/09_Docker_Object_Types_and_Filesystem.md)  ← NEW
# - [10. Container Interaction with Host](./Unit1/10_Container_Interaction_with_Host.md)     ← NEW
#
# ============================================================
# UPDATED README SECTION — Unit II table (replace existing)
# ============================================================
#
# ## 🗂️ Unit II — Topics Covered
#
# - [1. Dockerfile & Image Creation](./Unit2/01_Dockerfile_Image_Creation.md)           ← UPDATED
# - [2. Docker Networking](./Unit2/02_Docker_Networking.md)                             ← UPDATED
# - [3. Docker Storage — Volumes & Bind Mounts](./Unit2/03_Docker_Storage.md)           ← UPDATED
# - [4. Environment Variables](./Unit2/04_Environment_Variables.md)
# - [5. Registries — DockerHub, GHCR, Private](./Unit2/05_Registries.md)
# - [6. GHCR & Private Registries — Step-by-Step](./Unit2/06_GHCR_and_Private_Registries.md) ← NEW

# ============================================================
# TERMINAL COMMANDS (if doing it manually)
# ============================================================

# Navigate to your cloned repo
cd 2OM59Devops

# Copy all new/updated files into the repo
# (from wherever you saved the NEW-NOTES folder)

cp NEW-NOTES/Unit1/09_Docker_Object_Types_and_Filesystem.md ./Unit1/
cp NEW-NOTES/Unit1/10_Container_Interaction_with_Host.md ./Unit1/

# For Unit2 — rename old Dockerfile file if needed:
mv Unit2/01_Dockerfile_Basics.md Unit2/01_Dockerfile_Image_Creation.md

cp NEW-NOTES/Unit2/01_Dockerfile_Image_Creation.md ./Unit2/
cp NEW-NOTES/Unit2/02_Docker_Networking.md ./Unit2/
cp NEW-NOTES/Unit2/03_Docker_Storage.md ./Unit2/
cp NEW-NOTES/Unit2/06_GHCR_and_Private_Registries.md ./Unit2/

# Stage all changes
git add .

# Commit
git commit -m "feat: Add new Unit 1 & 2 notes — Object Types, GHCR, Container Interaction, updated Networking/Storage/Dockerfile"

# Push
git push origin main

# ============================================================
# DAILY UPDATE TEMPLATE (use after every class)
# ============================================================

# cd 2OM59Devops
# git add .
# git commit -m "notes: Add [TOPIC] — [DATE e.g. 2026-04-10]"
# git push origin main

# ============================================================
# ADDING SCREENSHOTS (after each lab)
# ============================================================

# cp ~/Desktop/your_screenshot.png ./Screenshots/2026-04-10_ghcr_push.png
# git add Screenshots/
# git commit -m "screenshots: GHCR push + pull practice — 2026-04-10"
# git push origin main
