# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

![screenshot](screenshots/ss31.png)

---

#### Screenshot 2 — Output of `ip a`

![screenshot](screenshots/ss32.png)

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

![screenshot](screenshots/ss33.png)

---

#### Screenshot 4 — Output of `sudo ufw status`

![screenshot](screenshots/ss34.png)

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

I allowed inbound HTTP access all ips, also to listen on port 80, while setting up security rules the ec2 server. The port process, users:(("nginx",pid=44240,fd=5),("nginx",pid=44239,fd=5),("nginx",pid=44238,fd=5))

---

**2. What proves SSH is active on port 22?**

I allowed inbound access all ips, also ssh to connect on port 22, while setting up security rules the ec2 server. Port process users:(("sshd",pid=23372,fd=3),("systemd",pid=1,fd=200))    

---

**3. Did you find any unexpected open ports? Explain briefly.**

No unexpected port was open while setting up inbound rule. 

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

![screenshot](screenshots/ss41.png)

---

#### Screenshot 2 — Output of `sudo nginx -t`

![screenshot](screenshots/ss42.png)

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

![screenshot](screenshots/ss43.png)

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

I will check Nginx status

---

**2. What's your basic rollback plan?**

If Nginx is not enabled, I will run sudo systemctl enable nginx 

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

![screenshot](screenshots/ss44.png)

![screenshot](screenshots/ss44a.png)

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

![screenshot](screenshots/ss45.png)

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

![screenshot](screenshots/ss46.png)

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

NO

---

**2. If there were no errors, what does that indicate about the system?**

No error, means no error was logged during the run time of the application at the moment the check was carried out.


---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

That the inbound traffic is allowed

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

![screenshot](screenshots/ss47.png)

---

#### Screenshot 2 — Output of `free -h`

![screenshot](screenshots/ss48.png)

---

#### Screenshot 3 — Output of `df -h`

![screenshot](screenshots/ss49.png)

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

![screenshot](screenshots/new.png)

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

Disk space, Monitoring it ensures our website has sufficient room to store and serve all its content without interruption. Keeping track of available storage helps us maintain performance, prevent downtime, and plan for future growth.

---

**2. What happens if disk becomes 100% full in a production server?**

We will expirence downtime and low perfomance. 

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

![screenshot](screenshots/ss50.png)

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

![screenshot](screenshots/ss51.png)

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

![screenshot](screenshots/ss52.png)

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

I can see my name edited in the App.js file. "Deployed by: ", "FOLARIN WALE", and "Date: ", "16/07/2026" — confirming the exact build and who deployed it.

---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

![screenshot](screenshots/ss53.png)

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

![screenshot](screenshots/ss54.png)

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![screenshot](screenshots/ss55.png)

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

Syntax error in the config file

---

**2. How did you fix the issue?**

I ran Nginx configuration test, nginx return with syntax error. I read through the config file again and found the error made corrections.

---

**3. How can you avoid this kind of issue in real production systems?**

Always, double check code for error.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

![screenshot](screenshots/ss56.png)

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![screenshot](screenshots/ss57.png)

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

Created an empty directory with no content, causing Nginx to return a 500 Internal Server Error since it had no files to serve.

---

**2. How did you fix the issue and restore the application?**

React build files were restored back into the newly created /var/www/html directory, restoring the content Nginx needed to serve

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

Never move or delete the web root directly in production environment

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

Because the key is stored local and can only be access locally

---

**2. Why should only required ports be open on a production server?**

To ensuring the server is only exposes to what is absolutely necessary function.

---

**3. Why is it important for Nginx to be enabled on boot?**

This ensures your website stays online and available to users at all times without you having to SSH in and manually start it every time.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Codes and application can be hacked 

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

To save cost

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/wale-folarin-956b6022a_devops-linux-ubuntu-ugcPost-7484280714356596736-EPU9/?utm_source=share&utm_medium=member_desktop&rcm=ACoAADl6z1IBZjWVdPX--51VXY7TxU7dXOVzE3c

---

#### Screenshot — Published LinkedIn post

![screenshot](screenshots/11.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [ ] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [ ] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [ ] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [ ] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*