# MongoDB
1. users Collection:
   {
  _id: ObjectId("65e23a8b1234567890abcdef"), 
  name: "John Doe",
  email: "john.doe@example.com",
  password: "hashedpassword123", 
  batch: "FSD-MERN-Stack-Oct2023",
  mentor_id: ObjectId("65e23a8b1234567890abcdef"), 
  created_at: ISODate("2023-09-01T10:00:00Z"),
  // Embedded attendance and codekata for quick access, or keep separate if very large
  attendance: [
    { date: ISODate("2023-10-01T09:00:00Z"), present: true },
    { date: ISODate("2023-10-02T09:00:00Z"), present: false },
    // ... more attendance records
   
  ],
  codekata_problems: [
    { problem_name: "Array Sum", solved_at: ISODate("2023-10-05T14:30:00Z") },
    { problem_name: "String Reverse", solved_at: ISODate("2023-10-06T11:00:00Z") },
    // ... more solved problems
    
  ],
  tasks_submitted: [
    {
      task_id: ObjectId("65e23a8b1234567890fdecba"), // Reference to task
      submission_date: ISODate("2023-10-10T17:00:00Z"),
      is_submitted: true
    },
    {
      task_id: ObjectId("65e23a8b1234567890fedcba"), // Reference to task
      submission_date: null,
      is_submitted: false
    }
    
  ]
}

2. mentors Collection:

   {
  _id: ObjectId("65e23a8b1234567890abcdef"),
  name: "Jane Smith",
  email: "jane.smith@example.com",
  
}

3.topics Collection:

{
  _id: ObjectId("65e23a8b1234567890fdecba"),
  name: "Introduction to HTML",
  date_taught: ISODate("2023-10-01T10:00:00Z"),
  tasks: [
  
    {
      task_name: "Build a static webpage",
      due_date: ISODate("2023-10-05T23:59:59Z")
    },
    {
      task_name: "HTML forms exercise",
      due_date: ISODate("2023-10-07T23:59:59Z")
    }
  ]
}

4.company_drives Collection:

{
  _id: ObjectId("65e23a8b1234567890abcde1"),
  company_name: "Google",
  drive_date: ISODate("2020-10-20T10:00:00Z"),
  package_offered: 15.5, 
  students_appeared: [
    { user_id: ObjectId("65e23a8b1234567890abcdef"), got_placement: true },
    { user_id: ObjectId("65e23a8b1234567890abcdef"), got_placement: false }

  ]
}

queries:

1. Find all the topics and tasks which are thought in the month of October.
   
   db.topics.aggregate([
  {
    $match: {
      $expr: {
        $eq: [{ $month: "$date_taught" }, 10] 
      }
    }
  },
  {
    $project: {
      _id: 0,
      topic_name: "$name",
      tasks: "$tasks.task_name" 
    }
}


2. Find all the company drives which appeared between 15 oct-2020 and 31-oct-2020.
 
   db.company_drives.find({
  drive_date: {
    $gte: ISODate("2020-10-15T00:00:00Z"),
    $lte: ISODate("2020-10-31T23:59:59Z")
  }
}, { company_name: 1, drive_date: 1, _id: 0 });

3. Find all the company drives and students who appeared for the placement.
 
   db.company_drives.aggregate([
{ 
    $unwind: "$students_appeared" 
   },
  {
    $match: {
      "students_appeared.appeared_for_placement": true 
    }
  },
  {
    $lookup: {
      from: "users",
      localField: "students_appeared.user_id",
      foreignField: "_id",
      as: "student_info"
    }
  },
  {
    $unwind: "$student_info" 
  },
  {
    $project: {
      _id: 0,
      company_name: "$company_name",
      drive_date: "$drive_date",
      student_name: "$student_info.name",
      student_email: "$student_info.email"
    }
  }

]);

4. Find the number of problems solved by the user in codekata.
 
   db.users.aggregate([
  {
    $project: {
      _id: 0,
      user_name: "$name",
      problems_solved: { $size: "$codekata_problems" }
    }

]);


5.Find all the mentors with who has the mentee's count more than 15.

db.mentors.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "_id",
      foreignField: "mentor_id",
      as: "mentees"
    }
  },
  {
    $project: {
      mentor_name: "$name",
      mentee_count: { $size: "$mentees" }
    }
  },
  {
    $match: {
      mentee_count: { $gt: 15 }
    }
  }
  
]);


 6.Find the number of users who are absent and task is not submitted between 15 oct-2020 and 31-oct-2020.

 db.users.aggregate([
  {
    $addFields: {
      absent_in_period: {
        $filter: {
          input: "$attendance",
          as: "att",
          cond: {
            $and: [
              { $eq: ["$$att.present", false] },
              { $gte: ["$$att.date", ISODate("2020-10-15T00:00:00Z")] },
              { $lte: ["$$att.date", ISODate("2020-10-31T23:59:59Z")] }
            ]
          }
        }
      },
      tasks_not_submitted_in_period: {
        $filter: {
          input: "$tasks_submitted",
          as: "ts",
          cond: {
            $eq: ["$$ts.is_submitted", false]
            
          }
        }
      }
    }
  },
  {
    $match: {
      "absent_in_period.0": { $exists: true }, 
      "tasks_not_submitted_in_period.0": { $exists: true } 
    }
  },

  {
    $count: "total_users_absent_and_task_not_submitted"
  }
]);







