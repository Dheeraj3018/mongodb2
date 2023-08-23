new Date("2020-10-31");
ISODate("2020-10-31T00:00:00Z");

// 1st.Query

db.topics
  .find({
    $and: [
      {
        topic_date: {
          $gte: ISODate("2020-10-15T00:00:00Z"),
          $lte: ISODate("2020-10-31T00:00:00Z"),
        },
      },
      {
        "task.task_date": {
          $gte: ISODate("2020-10-15T00:00:00Z"),
          $lte: ISODate("2020-10-31T00:00:00Z"),
        },
      },
    ],
  })
  .pretty();

// 2nd.Query

db.company_drives
  .find({
    drive_date: {
      $gte: ISODate("2020-10-15T00:00:00Z"),
      $lte: ISODate("2020-10-31T00:00:00Z"),
    },
  })
  .pretty();

// 3rd.Query

db.users
  .aggregate([
    {
      $lookup: {
        from: "company_drives",
        localField: "company_drives",
        foreignField: "company_id",
        as: "company_drives",
      },
    },
    {
      $project: {
        name: true,
        _id: false,
        email: true,
        "company_drives.company": true,
      },
    },
  ])
  .pretty();

// 4th.Query

db.users.aggregate([
  {
    $lookup: {
      from: "codekata",
      localField: "codekata_problems",
      foreignField: "q_id",
      as: " total_problems",
    },
  },
  {
    $project: {
      name: true,
      _id: false,
      "codekata.category": true,
      total_problems: {
        $cond: {
          if: { $isArray: "$codekata_problems" },
          then: { $size: "$codekata_problems" },
          else: "0",
        },
      },
    },
  },
]);

// 5th.Query

db.mentors
  .find({
    mentee_count: { $gte: 15 },
  })
  .pretty();

// 6th.Query

db.users
  .find({
    $or: [
      { absent_on: { $exists: true, $ne: [] } },
      { submitted_task: { $exists: true, $ne: [] } },
    ],
  })
  .count();
