React Waterfall Concept

The “waterfall” concept in React refers to a situation where multiple asynchronous calls (like API requests) depend on each other and execute sequentially instead of in parallel. This can cause performance issues as each request has to wait for the previous one to complete before starting.

How the Waterfall Happens

A common waterfall scenario in React occurs when: 1. A component fetches data in useEffect after mounting. 2. Another fetch depends on the first fetch’s results. 3. More nested fetches depend on previous ones.

Example:

import { useState, useEffect } from "react";

const WaterfallExample = () => {
const [data1, setData1] = useState(null);
const [data2, setData2] = useState(null);

useEffect(() => {
fetch("/api/data1")
.then((res) => res.json())
.then((result1) => {
setData1(result1);
return fetch(`/api/data2?id=${result1.id}`); // Next request depends on first
})
.then((res) => res.json())
.then(setData2);
}, []);

return <div>{data2 ? <p>{data2.info}</p> : "Loading..."}</div>;
};

Why is this bad?
• Each API request waits for the previous one, slowing down the overall load time.
• The UI remains in a loading state longer than necessary.
• Multiple dependent fetch calls create a performance bottleneck.

How to Fix React Waterfall?

To avoid the waterfall effect: 1. Parallel Fetching: Fetch multiple APIs simultaneously. 2. Use Promise.all(): Allows multiple promises to resolve together. 3. Pre-fetch Data: Fetch data at a higher level and pass it as props. 4. Server-Side Rendering (SSR): Fetch data before rendering the page.

Optimized Parallel Fetching Example

useEffect(() => {
Promise.all([fetch("/api/data1"), fetch("/api/data2")])
.then(([res1, res2]) => Promise.all([res1.json(), res2.json()]))
.then(([result1, result2]) => {
setData1(result1);
setData2(result2);
});
}, []);

getServerSideProps in Next.js

What is getServerSideProps?

getServerSideProps is a Next.js function that runs on the server before rendering a page. It allows fetching data at request time and sends it as props to the page component.

Key Features
• Runs only on the server (not in the browser).
• Fetches data on each request, making it suitable for dynamic content.
• Improves SEO because data is already available when the page is rendered.
• Cannot be used in API routes or components, only inside page components.

Basic Syntax

export async function getServerSideProps(context) {
const res = await fetch("https://api.example.com/data");
const data = await res.json();

return {
props: { data }, // Will be passed to the page component as props
};
}

const Page = ({ data }) => {
return <div>{data.title}</div>;
};

export default Page;

How getServerSideProps Works 1. The user requests a page. 2. Next.js runs getServerSideProps on the server. 3. It fetches data from an API or database. 4. The data is returned as props and sent to the page component. 5. The page is pre-rendered with the fetched data.

When to Use getServerSideProps

✅ Use when:
• You need fresh data on every request.
• The data depends on the user’s request or authentication.
• SEO is important and you want search engines to see pre-rendered content.

❌ Avoid when:
• The data doesn’t change often (use getStaticProps instead).
• The page needs fast responses (SSR can be slow).
• You need to fetch data in a client-side component.

Example: Fetching Data Based on Query Params

export async function getServerSideProps({ query }) {
const res = await fetch(`https://api.example.com/data?id=${query.id}`);
const data = await res.json();

return { props: { data } };
}

const Page = ({ data }) => <div>{data.name}</div>;

export default Page;

🔹 Dynamic behavior: query.id comes from the URL, so each request gets different data.

Differences: getServerSideProps vs. getStaticProps

Feature getServerSideProps getStaticProps
Runs on Server (on request) Server (at build time)
Data Freshness Always fresh Cached until next build
Performance Slower Faster (pre-built)
Best for Dynamic content (e.g., user data, dashboards) Static pages (e.g., blog, docs)

Let me know if you need a deeper explanation! 🚀
