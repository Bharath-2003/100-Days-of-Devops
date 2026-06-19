# Usage Guide - 100 Days of DevOps

## 🚀 Getting Started

This repository is now set up and ready for you to start documenting your DevOps learning journey!

## 📁 Repository Structure

```
100-days-of-devops/
├── README.md              # Main overview and progress tracker
├── USAGE_GUIDE.md         # This file - how to use the repository
├── .gitignore             # Git ignore rules
├── days/                  # All daily content goes here
│   └── day-001/          # Example day structure
│       ├── notes.md      # Your learning notes
│       ├── answers.md    # Exercise solutions
│       └── resources.md  # Additional resources
└── templates/            # Templates for new days
    ├── notes-template.md
    ├── answers-template.md
    └── resources-template.md
```

## 📝 Daily Workflow

### Step 1: Create a New Day Directory
```bash
cd 100-days-of-devops
mkdir -p days/day-XXX  # Replace XXX with day number (e.g., day-002)
```

### Step 2: Copy Templates
```bash
cp templates/notes-template.md days/day-XXX/notes.md
cp templates/answers-template.md days/day-XXX/answers.md
cp templates/resources-template.md days/day-XXX/resources.md
```

### Step 3: Fill in Your Content
Open each file and replace the placeholders with your actual content:
- **notes.md**: Document key concepts, commands, and takeaways
- **answers.md**: Write solutions to exercises and labs
- **resources.md**: Add links to documentation, articles, and tools

### Step 4: Commit Your Changes
```bash
git add days/day-XXX/
git commit -m "Add Day XXX: [Topic Name]"
```

## 💡 Tips for Effective Learning Documentation

### For Notes (notes.md)
- Write in your own words to reinforce learning
- Include code snippets with explanations
- Add diagrams or ASCII art for complex concepts
- Note down questions that arise during learning

### For Answers (answers.md)
- Document the problem statement clearly
- Explain your thought process
- Include step-by-step solutions
- Add troubleshooting notes if you encountered issues

### For Resources (resources.md)
- Bookmark official documentation
- Save helpful articles and tutorials
- Note down useful tools and their installation links
- Keep track of related topics to explore later

## 🔄 Git Commands Reference

### Daily Commits
```bash
# Check status
git status

# Add all changes
git add .

# Commit with message
git commit -m "Add Day XXX: Topic Name"

# View commit history
git log --oneline
```

### Pushing to Remote (Optional)
If you want to backup to GitHub/GitLab:
```bash
# Add remote repository
git remote add origin <your-repo-url>

# Push to remote
git push -u origin main
```

## 📊 Tracking Progress

Update the progress tracker in [`README.md`](README.md) as you complete each day:

```markdown
| Day | Topic | Status |
|-----|-------|--------|
| 001 | Introduction to DevOps | ✅ Completed |
| 002 | Linux Basics | ⏳ In Progress |
| 003 | Git & Version Control | ⏳ Pending |
```

Status Icons:
- ✅ Completed
- ⏳ In Progress
- 📝 Pending
- 🔄 Reviewing

## 🎯 Quick Commands

### Create and Setup New Day
```bash
# One-liner to create a new day
DAY=002 && mkdir -p days/day-$DAY && cp templates/*.md days/day-$DAY/ && cd days/day-$DAY && mv notes-template.md notes.md && mv answers-template.md answers.md && mv resources-template.md resources.md
```

### Search Your Notes
```bash
# Search for a specific term across all days
grep -r "docker" days/

# Search in specific file type
grep -r "kubernetes" days/*/notes.md
```

## 🔍 Finding Information Later

### By Topic
Use the README.md progress tracker to quickly locate topics

### By Command
Search through your notes:
```bash
grep -r "kubectl" days/
```

### By Date
Days are organized chronologically in the `days/` directory

## 📚 Best Practices

1. **Commit Daily**: Make it a habit to commit your work each day
2. **Be Consistent**: Use the templates to maintain consistency
3. **Add Context**: Always explain why, not just what
4. **Link Related Topics**: Cross-reference related days in your notes
5. **Review Regularly**: Revisit previous days to reinforce learning

## 🆘 Need Help?

- Check the templates in the `templates/` directory for structure guidance
- Review Day 001 as an example
- Keep your notes simple and focused on what you learned

---

**Happy Learning! 🚀**

Remember: The goal is to create a personal knowledge base that you can reference in the future. Make it work for you!