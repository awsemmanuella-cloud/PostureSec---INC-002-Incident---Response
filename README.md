# PostureSec — INC-002 Incident Response

**Incident ID:** INC-002  
**Title:** Application Became Slow and Unstable  
**Environment:** AWS Cloud Server (War Room Lab)  
**Date:** October 2026  
**Engineer:** Emmanuella Blessing Udeh  
**Role:** AI DevSecOps Engineering War Room  

---

## 1. What Was the Problem?

The PostureSec application suddenly became very slow.  

People trying to view posts were experiencing long waiting times and timeouts. A red warning banner appeared on the website saying a security incident had been detected (INC-002).

---

## 2. What Really Happened? (Simple Explanation)

As part of the training exercise, the system automatically created 300 test posts.

When someone tried to view the list of posts, the application tried to bring **all 300 posts at once**.

This is like trying to carry 300 books in your hands at the same time — it becomes heavy, slow, and difficult to manage.

Because of this, the application response became very large and took much longer than normal to load.

---

## 3. How We Investigated It

Here is what we did step by step:

1. We opened the application and saw the red incident warning.
2. We connected to the server.
3. We checked the incident status and confirmed it was active.
4. We looked at the system logs and saw that the application had become extremely slow.
5. We examined the application code that fetches the list of posts.
6. We discovered that the code had no limit — it was trying to return every post in the database.
7. We fixed the code by limiting it to the latest 50 posts only.
8. We restarted the application and tested it again.
9. We confirmed that the application became much faster and more stable.

---

## 4. Challenges We Faced and How We Solved Them

While working on this incident, we faced several practical challenges:

- Sometimes the system said “No such file or directory” because we were not in the correct folder. We fixed it by moving into the right project folder.
- The connection to the server kept dropping. We solved it by reconnecting properly with the correct key and address.
- There was confusion between the private address and the public address of the server. We learned that only the public address works in the browser.
- Some commands were typed incorrectly at first. We corrected the spelling and format of the commands.
- While editing the code, the file became messy. We cleaned it by carefully rewriting the correct version.
- At some point the logs were not showing. We used a different command to view them successfully.

These challenges are normal when working with real systems, and solving them is part of the learning process.

---

## 5. The Root Cause

The main problem was found in the part of the code responsible for showing the list of posts.

The code was asking the database for every single post without any limit.  

When the system created 300 test posts, the application tried to return all of them at once. This made the response slow and heavy.

---

## 6. How We Fixed It

We updated the code so that the application only returns the **latest 50 posts** instead of all posts.

This small change made a big difference. The application no longer tried to carry everything at once.

---

## 7. Results After the Fix

Before the fix:
- The application took about 3850 milliseconds to respond.
- The size of the response was about 12 MB.
- The system was clearly struggling.

After the fix:
- The response time dropped to about 244 milliseconds.
- The response size reduced to about 2 MB.
- The application remained much more stable even under the same load.

In simple terms, the application became significantly faster and lighter.

---

## 8. Security Thinking Behind This Incident

This incident was not caused by a hacker. It was caused by a design weakness that only appeared when the system was under pressure.

Important lessons:

- Security is not only about stopping attackers. It is also about making sure systems remain reliable when things get difficult.
- When an application has no limits, it can become slow or unavailable. In real life, this can frustrate users and damage trust.
- Good engineers investigate with evidence (logs, metrics, and code) instead of guessing.
- Simple protections like limiting how much data is returned at once are powerful ways to make systems stronger.
- Performance problems and security problems are often connected.

At PostureSec, we practice breaking things on purpose so we can learn how to build systems that do not break easily in the real world.

Security is not a tool. It is a mindset.

---

## 9. Final Status

- The root cause was identified.
- A clear fix was applied.
- The application performance improved significantly.
- The incident was fully investigated and documented.

- ---

## Code: Before the Fix (Problematic Version)

```js
router.get('/', async (req, res) => {
  const result = await pool.query(
    `SELECT p.*,
     (SELECT COUNT(*) FROM comments c WHERE c.id = p.id) as comment_count
     FROM posts p
     ORDER BY p.created_at DESC`
  );
  res.json(result.rows);
});

---Problem:
There was no limit. The application tried to return every post in the database at once.

Code: After the Fix:
router.get('/', async (req, res) => {
  try {
    const result = await pool.query(
      `SELECT p.*,
       (SELECT COUNT(*) FROM comments c WHERE c.post_id = p.id) as comment_count
       FROM posts p
       ORDER BY p.created_at DESC
       LIMIT 50`
    );
    res.json(result.rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Failed to fetch posts' });
  }
});

## What changed:
Added LIMIT 50 so the application only returns the latest 50 posts.
Corrected the comment count query to use the proper relationship (c.post_id = p.id).

**Built with discipline and curiosity during the PostureSec AI DevSecOps War Room.
Emmanuella Blessing Udeh | October 2026
