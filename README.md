# MongoDB

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







