1st Codebase (Using getServerSideProps)

How It Works
• Uses getServerSideProps, which runs on the server before rendering.
• Fetches courses from NEXT_URL/api/admin/courses/ on each request.
• The fetched courses are passed as props to the Courses component.
• Data is available before the page loads, improving SEO.
• Authentication via localStorage is commented out since localStorage is not available on the server.

Pros

✅ Faster initial page load since data is fetched before rendering.
✅ SEO-friendly since the data is ready when the page is rendered.
✅ No client-side fetching, reducing the risk of layout shifts.

Cons

❌ Every request makes a new API call, increasing server load.
❌ Local storage can’t be used for authentication in getServerSideProps, requiring cookies instead.
❌ Not suitable for static content that doesn’t change often.

2nd Codebase (Using useEffect and Client-Side Fetching)

How It Works
• The Courses component initially renders without data.
• useEffect runs after the component mounts, fetching courses from the API.
• The API call includes an Authorization header with a token from localStorage (possible only on the client-side).
• The fetched courses are stored in a useState variable (setCourses).

Pros

✅ Authentication using localStorage works, making it easier to manage sessions.
✅ No extra load on the server during SSR (less server-side work).
✅ Works better when combined with client-side interactivity (e.g., filtering or pagination).

Cons

❌ Initial load is slower since the page renders before fetching data.
❌ SEO is weaker because search engines see an empty page first.
❌ Causes a waterfall effect (empty UI → fetch data → update UI), leading to layout shifts.

Key Differences Summary

Feature 1st Codebase (getServerSideProps) 2nd Codebase (useEffect & useState)
Data Fetching Server-side before rendering Client-side after rendering
Performance Faster initial load Slower initial load
SEO Better SEO (data available at render time) Weaker SEO (data loads later)
Authentication Requires cookies (no localStorage) Uses localStorage for authentication
API Load Fetches data on every request Fetches data only after component mounts
UI Behavior Page loads with data immediately Page loads first, then updates with data

Which One Should You Use?

✅ Use getServerSideProps (1st Codebase) if:
• SEO is important (e.g., blog, marketing site).
• You want fresh data on every request.
• You don’t rely on localStorage for authentication.

✅ Use useEffect (2nd Codebase) if:
• The page is not SEO-critical (e.g., dashboards, admin panels).
• You need client-side authentication (localStorage).
• The data doesn’t need to be preloaded before rendering.

If you need both SEO and authentication, you can:
• Use getServerSideProps to fetch an authentication token from cookies.
• Use useEffect to fetch data after authentication.

Here’s a side-by-side comparison of the two approaches:

1️⃣ Using getServerSideProps (Server-Side Rendering)

import { Button, Card, Typography } from "@mui/material";
import { useRouter } from "next/router.js";
import type { Course } from "@/store/atoms/course.js";
import { NEXT_URL } from "@/config";
import axios from "axios";

function Courses({ courses }: { courses: Course[] }) {
return (

<div style={{ display: "flex", flexWrap: "wrap", justifyContent: "center" }}>
{courses.map((course) => {
return <Course key={course._id} course={course} />;
})}
</div>
);
}

function Course({ course }: { course: Course }) {
const router = useRouter();

return (
<Card style={{ margin: 10, width: 300, minHeight: 200, padding: 20 }}>
<Typography textAlign={"center"} variant="h5">{course.title}</Typography>
<Typography textAlign={"center"} variant="subtitle1">{course.description}</Typography>
<img src={course.imageLink} style={{ width: 300 }} alt={course.title} />

<div style={{ display: "flex", justifyContent: "center", marginTop: 20 }}>
<Button
variant="contained"
size="large"
onClick={() => router.push("/course/" + course.\_id)} >
Edit
</Button>
</div>
</Card>
);
}

export default Courses;

// Server-side data fetching
export async function getServerSideProps() {
console.log("Fetching courses on the server...");
const response = await axios.get(`${NEXT_URL}/api/admin/courses/`);

return {
props: {
courses: response.data.courses,
},
};
}

2️⃣ Using useEffect (Client-Side Fetching)

import { Button, Card, Typography } from "@mui/material";
import { useEffect, useState } from "react";
import axios from "axios";
import { useRouter } from "next/router.js";
import type { Course } from "@/store/atoms/course.js";
import { NEXT_URL } from "@/config";

function Courses() {
const [courses, setCourses] = useState<Course[]>([]);

useEffect(() => {
const fetchCourses = async () => {
const response = await axios.get(`${NEXT_URL}/api/admin/courses/`, {
headers: {
Authorization: `Bearer ${localStorage.getItem("token")}`,
},
});
setCourses(response.data.courses);
};

    fetchCourses();

}, []);

return (

<div style={{ display: "flex", flexWrap: "wrap", justifyContent: "center" }}>
{courses.map((course) => {
return <Course key={course._id} course={course} />;
})}
</div>
);
}

function Course({ course }: { course: Course }) {
const router = useRouter();

return (
<Card style={{ margin: 10, width: 300, minHeight: 200, padding: 20 }}>
<Typography textAlign={"center"} variant="h5">{course.title}</Typography>
<Typography textAlign={"center"} variant="subtitle1">{course.description}</Typography>
<img src={course.imageLink} style={{ width: 300 }} alt={course.title} />

<div style={{ display: "flex", justifyContent: "center", marginTop: 20 }}>
<Button
variant="contained"
size="large"
onClick={() => router.push("/course/" + course.\_id)} >
Edit
</Button>
</div>
</Card>
);
}

export default Courses;

Key Differences in Code

Feature getServerSideProps (SSR) useEffect (Client-side)
Where Data is Fetched On the server before rendering On the client after rendering
SEO Good (content is available at render time) Bad (page first renders empty)
Performance Faster initial load Slower initial load due to fetching after render
Authentication Needs cookies (no localStorage on the server) Can use localStorage
API Load Calls API on every request Calls API only when the component mounts
Page Load Behavior Loads fully populated Loads, then fetches data

When to Use Which Approach?

✅ Use getServerSideProps (SSR) when:
• You need SEO optimization.
• You want fresh data on every request.
• You don’t rely on localStorage for authentication.

✅ Use useEffect (Client-side) when:
• You need authentication using localStorage.
• SEO is not a priority (e.g., dashboards, admin panels).
• The data doesn’t need to be fresh on every request.

Let me know if you need any refinements or a hybrid approach! 🚀
