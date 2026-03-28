# 📸 Docker Hub Practice — Screenshots

This folder contains screenshots of hands-on Docker Hub activities done during class.

---

## How to Add Your Screenshots

1. Take a screenshot of your Docker Hub activity
2. Save the file with a descriptive name (e.g., `push_nginx_dockerhub.png`)
3. Place it in this `Screenshots/` folder
4. Update the table below with the filename and description

---

## Screenshot Log

| # | Screenshot File | What it shows | Date |
|---|----------------|---------------|------|
| 1 | _(add your file here)_ | _(describe what you did)_ | _(date)_ |
| 2 | | | |
| 3 | | | |

---

## Practice Activities to Screenshot

Use this checklist — screenshot each one as you complete it:

- [ ] Docker Hub account created / logged in
- [ ] Pulled an official image (e.g., `nginx`, `ubuntu`)
- [ ] `docker images` output showing pulled images
- [ ] Container running (`docker ps` output)
- [ ] Browser showing app running on `localhost:8080`
- [ ] Image tagged with your Docker Hub username
- [ ] Image pushed to Docker Hub repo
- [ ] Docker Hub repo page showing your pushed image
- [ ] Pulled your own image on a different container
- [ ] `docker stats` showing live resource usage
- [ ] MySQL container running with env variables

---

## Example Commands to Practice & Screenshot

```bash
# 1. Pull nginx and run it
docker pull nginx
docker run -d -p 8080:80 --name mynginx nginx
docker ps
# Screenshot: docker ps output

# 2. Open browser → localhost:8080
# Screenshot: browser showing nginx welcome page

# 3. Tag and push your image
docker tag nginx yourusername/my-nginx:v1
docker login
docker push yourusername/my-nginx:v1
# Screenshot: Docker Hub repo page showing the image

# 4. Pull from Docker Hub
docker pull yourusername/my-nginx:v1
# Screenshot: successful pull message

# 5. Clean up
docker stop mynginx
docker rm mynginx
```

---

> Add your screenshots here as you complete each lab session.  
> Filename format: `YYYY-MM-DD_activity_name.png`  
> Example: `2026-03-28_push_nginx_to_dockerhub.png`
