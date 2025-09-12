---
title: SQL Injection in PDO’s Prepared Statements
categories:
  - sqli
  - php
---

Code that looks correct but isn't always makes for very interesting case
studies. These are also fun or terrifying depending on whether you’re shipping
code or breaking code. Adam Kues’s excellent write-up shows exactly that: a
clever technique for exploiting SQL injections in PHP’s PDO despite using
prepared statements. PDO stands for PHP Data Objects and is a standard and
presumed safe way to abstract database access in PHP codebases. A must-read for
anyone who writes, audits, or relies on PHP.

[https://slcyber.io/assetnote-security-research-center/a-novel-technique-for-sql-injection-in-pdos-prepared-statements/](https://slcyber.io/assetnote-security-research-center/a-novel-technique-for-sql-injection-in-pdos-prepared-statements/)
