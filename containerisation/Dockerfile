# Phase 2 — custom static page served by Nginx
FROM nginx:alpine

# Replace the default Nginx welcome page with our GitOps demo page
COPY index.html /usr/share/nginx/html/index.html

# Nginx serves on port 80 inside the container
EXPOSE 80
