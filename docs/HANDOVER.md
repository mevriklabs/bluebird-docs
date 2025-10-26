# 🧭 Project Handover Document

## Overview

This document serves as the **technical handover guide** for the PHP-based communication platform built using **Laravel **, **PHP 8.2-FPM**, and **Nginx**.  
It outlines the core architecture, deployment process, environment configuration, and operational guidelines to help new developers quickly onboard and maintain the system.

---

## 🧱 Core Architecture

| Component | Description |
|------------|-------------|
| **Language** | PHP 8.2 (FPM) |
| **Framework** | Laravel |
| **Job Handling** | Laravel Horizon + Supervisor |
| **Cache & Queue Driver** | Redis |
| **Databases** | MySQL (Primary), PostgreSQL (Reports), ClickHouse (Static data, phone matching) |
| **Web Server** | Nginx |
| **Package Structure** | Built using **Loris Leiva Actions** for controller and job logic |

---

## 📂 Repository

**Repository URL:**  
`https://github.com/genexgit/mevrik-channels-php.git`

### Clone the Project

```bash
git clone https://github.com/genexgit/mevrik-channels-php.git
cd mevrik-channels-php
