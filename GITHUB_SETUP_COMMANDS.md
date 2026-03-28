# 🚀 GitHub Setup Commands — INT332 DevOps Notes
# Run these commands IN ORDER to set up your repo from scratch.
# Give this file to GitHub Copilot with the instruction:
# "Execute these commands to set up my DevOps notes repo"

# ============================================================
# STEP 1 — Configure Git (only needed once on your machine)
# ============================================================
git config --global user.name "bhoomiijain"
git config --global user.email "your-email@example.com"
# ⬆️ Replace with your actual GitHub email

# ============================================================
# STEP 2 — Clone your existing GitHub repo
# ============================================================
git clone https://github.com/bhoomiijain/2OM59Devops.git
cd 2OM59Devops

# ============================================================
# STEP 3 — Create the folder structure
# ============================================================
mkdir -p Unit1
mkdir -p Unit2
mkdir -p Screenshots

# ============================================================
# STEP 4 — Copy all note files into the repo
# (Run this from wherever you saved the DevOps-Notes folder)
# ============================================================

# If you downloaded the files from Claude, copy them:
cp /path/to/DevOps-Notes/README.md .
cp /path/to/DevOps-Notes/Unit1/*.md ./Unit1/
cp /path/to/DevOps-Notes/Unit2/*.md ./Unit2/
cp /path/to/DevOps-Notes/Screenshots/README.md ./Screenshots/

# ============================================================
# STEP 5 — Add a .gitignore file
# ============================================================
cat > .gitignore << 'EOF'
*.DS_Store
.env
*.log
__pycache__/
node_modules/
Thumbs.db
EOF

# ============================================================
# STEP 6 — Stage all files
# ============================================================
git add .

# ============================================================
# STEP 7 — Commit with a message
# ============================================================
git commit -m "feat: Add Unit 1 & Unit 2 notes — Docker DevOps INT332

- Added README with course structure
- Unit 1: DevOps basics, Virtualization, Containerization,
  Container Runtime, Namespaces, cgroups, Images, Registries,
  Docker Architecture & Basic Commands
- Unit 2: Dockerfile, Networking, Storage, Env Variables, Registries
- Screenshots folder with practice checklist"

# ============================================================
# STEP 8 — Push to GitHub
# ============================================================
git push origin main
# If your branch is called master: git push origin master

# ============================================================
# DONE ✅
# Your repo should now be visible at:
# https://github.com/bhoomiijain/2OM59Devops
# ============================================================


# ============================================================
# DAILY UPDATE COMMANDS (use these every day after class)
# ============================================================

# Navigate to your repo
cd 2OM59Devops

# Check what changed
git status

# Stage changes
git add .

# Commit with today's date and topic
git commit -m "notes: Add [TOPIC NAME] — [DATE]
# Example: git commit -m "notes: Add Docker Volumes & Bind Mounts — 2026-03-28"

# Push
git push origin main


# ============================================================
# ADDING SCREENSHOTS (after each lab session)
# ============================================================

# Copy screenshot to Screenshots folder
cp ~/Downloads/your_screenshot.png ./Screenshots/2026-03-28_activity_name.png

# Stage + commit + push
git add Screenshots/
git commit -m "screenshots: Add Docker Hub push practice — 2026-03-28"
git push origin main


# ============================================================
# COPILOT INSTRUCTIONS
# ============================================================
# If using GitHub Copilot CLI or chat, paste this prompt:
#
# "I have a DevOps notes repo at https://github.com/bhoomiijain/2OM59Devops.git
#  Please help me:
#  1. Create folders: Unit1/, Unit2/, Screenshots/
#  2. Add the markdown files I provide into the correct folders
#  3. Commit with the message: 'feat: Add Unit 1 & 2 notes INT332'
#  4. Push to the main branch"
