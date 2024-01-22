### Design database for Zen class programme mongoDb

# users
# codekata
# attendance
# topics
# tasks
# company_drives
# mentors
---
```js
 
// Users collection
>db.users.insertMany([
  { user_id: 1, name: "sudharshan", email: "sudharshan1903@gmail.com" },
  { user_id: 2, name: "ram", email: "ram@example.com" },
  { user_id: 3, name: "kumar", email: "kumar@example.com" },
  { user_id: 4, name: "dinesh", email: "dinesh@example.com" },
  { user_id: 5, name: "james", email: "james@example.com" },
]);

// Codekata collection
>db.codekata.insertMany([
  { codekata_id: 1, name: "React Basics", difficulty: "Easy" },
  { codekata_id: 2, name: "Python Advanced", difficulty: "Hard" },
  { codekata_id: 3, name: "JavaScript Fundamentals", difficulty: "Medium" },
  { codekata_id: 4, name: "Algorithms in java", difficulty: "Hard" },
  { codekata_id: 5, name: "Data Structures in c++", difficulty: "Medium" },
]);

// Attendance collection
>db.attendance.insertMany([
  { user_id: 1, date: ISODate("2020-10-15"), status: "Present" },
  { user_id: 2, date: ISODate("2020-10-20"), status: "Absent" },
  { user_id: 3, date: ISODate("2020-10-25"), status: "Present" },
  { user_id: 4, date: ISODate("2020-10-28"), status: "Absent" },
  { user_id: 5, date: ISODate("2020-10-30"), status: "Present" },
]);

// Topics collection
>db.topics.insertMany([
  { topic_id: 1, name: "Data Structures" },
  { topic_id: 2, name: "Algorithms" },
  { topic_id: 3, name: "Database Design" },
  { topic_id: 4, name: "Web Development Basics" },
  { topic_id: 5, name: "Artificial Intelligence Concepts" },
]);

// Tasks collection
>db.tasks.insertMany([
  { task_id: 1, user_id: 1, task_name: "TodoLIST", status: "Submitted" },
  { task_id: 2, user_id: 2, task_name: "TicTacToe", status: "Not Submitted" },
  { task_id: 3, user_id: 3, task_name: "shopping", status: "Submitted" },
  { task_id: 4, user_id: 4, task_name: "user management", status: "Not Submitted" },
  { task_id: 5, user_id: 5, task_name: "library management", status: "Submitted" },
]);

// Company Drives collections
>db.company_drives.insertMany([
  { drive_id: 1, name: "zoho Corp", date: ISODate("2020-10-25") },
  { drive_id: 2, name: "hcl", date: ISODate("2020-10-28") },
  { drive_id: 3, name: "L&T", date: ISODate("2020-10-20") },
  { drive_id: 4, name: "accenture", date: ISODate("2020-10-30") },
  { drive_id: 5, name: "techni color", date: ISODate("2020-10-31") },
]);

// Mentors collection
>db.mentors.insertMany([
  { mentor_id: 1, name: "pugazh", mentee_count: 20 },
  { mentor_id: 2, name: "sangeetha", mentee_count: 10 },
  { mentor_id: 3, name: "arun", mentee_count: 25 },
  { mentor_id: 4, name: "sweetha", mentee_count: 18 },
  { mentor_id: 5, name: "vino", mentee_count: 30 },
]);

```
### 1.Find all the topics and tasks which are thought in the month of October
```js
>db.topics.find();
>db.tasks.find({ created_at: { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") } });
```
### 2.Find all the company drives which appeared between 15 oct-2020 and 31-oct-2020
```js
>db.company_drives.find({ date: { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") } });

```
### 3.Find all the company drives and students who are appeared for the placement.
```js
>db.company_drives.aggregate([
  {
    $lookup: {
      from: "attendance",
      localField: "drive_id",
      foreignField: "drive_id",
      as: "drive_attendance"
    }
  },
  {
    $lookup: {
      from: "users",
      localField: "drive_attendance.user_id",
      foreignField: "user_id",
      as: "students"
    }
  },
  {
    $project: {
      _id: 0,
      drive_id: 1,
      name: 1,
      date: 1,
      students: {
        user_id: 1,
        name: 1,
        email: 1
      }
    }
  }
]);

```
### 4.Find the number of problems solved by the user in codekata
```js
>db.tasks.aggregate([
  {
    $match: { status: "Submitted" }
  },
  {
    $group: {
      _id: "$user_id",
      problems_solved: { $sum: 1 }
    }
  },
  {
    $lookup: {
      from: "users",
      localField: "_id",
      foreignField: "user_id",
      as: "user_info"
    }
  },
  {
    $project: {
      _id: 0,
      user_id: "$_id",
      user_name: "$user_info.name",
      problems_solved: 1
    }
  }
]);

```
### 5. Find all the mentors with who has the mentee's count more than 15
```js
>db.mentors.find({ mentee_count: { $gt: 15 } });

```
### 6.Find the number of users who are absent and task is not submitted between 15 oct-2020 and 31-oct-2020
```js
>db.attendance.aggregate([
  {
    $match: {
      date: { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") },
      status: "Absent"
    }
  },
  {
    $lookup: {
      from: "tasks",
      localField: "user_id",
      foreignField: "user_id",
      as: "task_info"
    }
  },
  {
    $match: {
      "task_info.status": { $ne: "Submitted" }
    }
  },
  {
    $group: {
      _id: null,
      users_count: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      users_count: 1
    }
  }
]);

```