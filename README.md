# Front-End-Developer-Role

Here is my Answers with source code.
Q1.
Backend Code (Node.js + Express)

const express = require(&#39;express&#39;);
const mongoose = require(&#39;mongoose&#39;);
const cors = require(&#39;cors&#39;);

const app = express();
const PORT = 5000;

// Middleware
app.use(express.json());
app.use(cors());

// MongoDB connection
mongoose.connect(&#39;mongodb://localhost:27017/user_availability&#39;, {
useNewUrlParser: true,
useUnifiedTopology: true,
});

const availabilitySchema = new mongoose.Schema({
user: { type: String, required: true },
start: { type: Date, required: true },
end: { type: Date, required: true },
duration: { type: Number, required: true },
scheduledSlots: { type: Array, default: [] },
});

const Availability = mongoose.model(&#39;Availability&#39;, availabilitySchema);

// Routes

// Add availability
app.post(&#39;/api/availability&#39;, async (req, res) =&gt; {
try {
const { user, start, end, duration } = req.body;
const availability = new Availability({ user, start, end, duration });
await availability.save();
res.status(201).send(availability);
} catch (error) {
res.status(400).send(error);
}
});

// Get availability by user
app.get(&#39;/api/availability/:user&#39;, async (req, res) =&gt; {
try {
const { user } = req.params;
const availability = await Availability.find({ user });
res.status(200).send(availability);
} catch (error) {
res.status(400).send(error);
}
});

// Update availability
app.put(&#39;/api/availability/:id&#39;, async (req, res) =&gt; {
try {
const { id } = req.params;
const updates = req.body;
const availability = await Availability.findByIdAndUpdate(id, updates, { new: true });

res.status(200).send(availability);
} catch (error) {
res.status(400).send(error);
}
});

// Delete availability
app.delete(&#39;/api/availability/:id&#39;, async (req, res) =&gt; {
try {
const { id } = req.params;
await Availability.findByIdAndDelete(id);
res.status(200).send({ message: &#39;Availability deleted successfully&#39; });
} catch (error) {
res.status(400).send(error);
}
});

// Start server
app.listen(PORT, () =&gt; {
console.log(`Server running on http://localhost:${PORT}`);
});

// Frontend Code (React)

import React, { useState, useEffect } from &#39;react&#39;;
import axios from &#39;axios&#39;;
import Calendar from &#39;react-calendar&#39;;
import &#39;react-calendar/dist/Calendar.css&#39;;

const App = () =&gt; {
const [availability, setAvailability] = useState([]);
const [user, setUser] = useState(&#39;user@gmail.com&#39;);
const [selectedDate, setSelectedDate] = useState(new Date());

const [start, setStart] = useState(&#39;&#39;);
const [end, setEnd] = useState(&#39;&#39;);
const [duration, setDuration] = useState(30);

// Fetch user availability
useEffect(() =&gt; {
const fetchAvailability = async () =&gt; {
const response = await axios.get(`/api/availability/${user}`);
setAvailability(response.data);
};

fetchAvailability();
}, [user]);

// Add availability
const handleAddAvailability = async () =&gt; {
const newAvailability = {
user,
start: new Date(`${selectedDate.toISOString().split(&#39;T&#39;)[0]}T${start}`),
end: new Date(`${selectedDate.toISOString().split(&#39;T&#39;)[0]}T${end}`),
duration,
};

const response = await axios.post(&#39;/api/availability&#39;, newAvailability);
setAvailability([...availability, response.data]);
};

return (
&lt;div&gt;
&lt;h1&gt;Dynamic User Availability&lt;/h1&gt;
&lt;label&gt;Email:&lt;/label&gt;
&lt;input
type=&quot;email&quot;

value={user}
onChange={(e) =&gt; setUser(e.target.value)}
/&gt;
&lt;Calendar
onChange={(date) =&gt; setSelectedDate(date)}
value={selectedDate}
/&gt;
&lt;div&gt;
&lt;label&gt;Start Time:&lt;/label&gt;
&lt;input
type=&quot;time&quot;
value={start}
onChange={(e) =&gt; setStart(e.target.value)}
/&gt;
&lt;label&gt;End Time:&lt;/label&gt;
&lt;input
type=&quot;time&quot;
value={end}
onChange={(e) =&gt; setEnd(e.target.value)}
/&gt;
&lt;label&gt;Duration (minutes):&lt;/label&gt;
&lt;input
type=&quot;number&quot;
value={duration}
onChange={(e) =&gt; setDuration(e.target.value)}
/&gt;
&lt;button onClick={handleAddAvailability}&gt;Add Availability&lt;/button&gt;
&lt;/div&gt;
&lt;h2&gt;Availability&lt;/h2&gt;
&lt;ul&gt;
{availability.map((slot) =&gt; (
&lt;li key={slot._id}&gt;
{new Date(slot.start).toLocaleString()} - {new Date(slot.end).toLocaleString()} ({slot.duration} minutes)

&lt;/li&gt;
))}
&lt;/ul&gt;
&lt;/div&gt;
);
};

export default App;
Q2.
// Backend Code (Node.js + Express) with Admin Scheduling

const express = require(&#39;express&#39;);
const mongoose = require(&#39;mongoose&#39;);
const cors = require(&#39;cors&#39;);

const app = express();
const PORT = 5000;

// Middleware
app.use(express.json());
app.use(cors());

// MongoDB connection
mongoose.connect(&#39;mongodb://localhost:27017/user_availability&#39;, {
useNewUrlParser: true,
useUnifiedTopology: true,
});

const availabilitySchema = new mongoose.Schema({
user: { type: String, required: true },
start: { type: Date, required: true },
end: { type: Date, required: true },
duration: { type: Number, required: true },

scheduledSlots: { type: Array, default: [] },
});

const sessionSchema = new mongoose.Schema({
user: { type: String, required: true },
start: { type: Date, required: true },
end: { type: Date, required: true },
duration: { type: Number, required: true },
attendees: [{
name: String,
email: String,
}],
});

const Availability = mongoose.model(&#39;Availability&#39;, availabilitySchema);
const Session = mongoose.model(&#39;Session&#39;, sessionSchema);

// Routes

// Add availability
app.post(&#39;/api/availability&#39;, async (req, res) =&gt; {
try {
const { user, start, end, duration } = req.body;
const availability = new Availability({ user, start, end, duration });
await availability.save();
res.status(201).send(availability);
} catch (error) {
res.status(400).send(error);
}
});

// Get availability by user
app.get(&#39;/api/availability/:user&#39;, async (req, res) =&gt; {

try {
const { user } = req.params;
const availability = await Availability.find({ user });
res.status(200).send(availability);
} catch (error) {
res.status(400).send(error);
}
});

// Schedule session
app.post(&#39;/api/schedule&#39;, async (req, res) =&gt; {
try {
const { user, start, end, duration, attendees } = req.body;

// Check for conflicts
const conflicts = await Session.find({
user,
$or: [
{ start: { $lt: new Date(end) }, end: { $gt: new Date(start) } },
],
});

if (conflicts.length &gt; 0) {
return res.status(400).send({ message: &#39;Time slot conflict detected&#39; });
}

const session = new Session({ user, start, end, duration, attendees });
await session.save();
res.status(201).send(session);
} catch (error) {
res.status(400).send(error);
}
});

// Get all sessions for a user
app.get(&#39;/api/sessions/:user&#39;, async (req, res) =&gt; {
try {
const { user } = req.params;
const sessions = await Session.find({ user });
res.status(200).send(sessions);
} catch (error) {
res.status(400).send(error);
}
});

// Start server
app.listen(PORT, () =&gt; {
console.log(`Server running on http://localhost:${PORT}`);
});

// Frontend Code (React)

import React, { useState, useEffect } from &#39;react&#39;;
import axios from &#39;axios&#39;;
import Calendar from &#39;react-calendar&#39;;
import &#39;react-calendar/dist/Calendar.css&#39;;

const AdminDashboard = () =&gt; {
const [availability, setAvailability] = useState([]);
const [sessions, setSessions] = useState([]);
const [user, setUser] = useState(&#39;&#39;);
const [start, setStart] = useState(&#39;&#39;);
const [end, setEnd] = useState(&#39;&#39;);
const [duration, setDuration] = useState(30);
const [attendees, setAttendees] = useState([{ name: &#39;&#39;, email: &#39;&#39; }]);

// Fetch user availability
useEffect(() =&gt; {
if (user) {
const fetchAvailability = async () =&gt; {
const response = await axios.get(`/api/availability/${user}`);
setAvailability(response.data);
};

fetchAvailability();
}
}, [user]);

// Fetch user sessions
useEffect(() =&gt; {
if (user) {
const fetchSessions = async () =&gt; {
const response = await axios.get(`/api/sessions/${user}`);
setSessions(response.data);
};

fetchSessions();
}
}, [user]);

// Schedule session
const handleScheduleSession = async () =&gt; {
const newSession = {
user,
start: new Date(start),
end: new Date(end),
duration,
attendees,
};

try {
const response = await axios.post(&#39;/api/schedule&#39;, newSession);
setSessions([...sessions, response.data]);
} catch (error) {
alert(&#39;Conflict detected or invalid input&#39;);
}
};

return (
&lt;div&gt;
&lt;h1&gt;Admin Dashboard&lt;/h1&gt;
&lt;label&gt;Select User:&lt;/label&gt;
&lt;input
type=&quot;text&quot;
placeholder=&quot;Enter user email&quot;
value={user}
onChange={(e) =&gt; setUser(e.target.value)}
/&gt;

&lt;h2&gt;Availability&lt;/h2&gt;
&lt;ul&gt;
{availability.map((slot) =&gt; (
&lt;li key={slot._id}&gt;
{new Date(slot.start).toLocaleString()} - {new Date(slot.end).toLocaleString()} ({slot.duration} minutes)
&lt;/li&gt;
))}
&lt;/ul&gt;

&lt;h2&gt;Schedule a Session&lt;/h2&gt;
&lt;label&gt;Start Time:&lt;/label&gt;
&lt;input
type=&quot;datetime-local&quot;

value={start}
onChange={(e) =&gt; setStart(e.target.value)}
/&gt;

&lt;label&gt;End Time:&lt;/label&gt;
&lt;input
type=&quot;datetime-local&quot;
value={end}
onChange={(e) =&gt; setEnd(e.target.value)}
/&gt;

&lt;label&gt;Duration (minutes):&lt;/label&gt;
&lt;input
type=&quot;number&quot;
value={duration}
onChange={(e) =&gt; setDuration(e.target.value)}
/&gt;

&lt;h3&gt;Attendees&lt;/h3&gt;
{attendees.map((attendee, index) =&gt; (
&lt;div key={index}&gt;
&lt;input
type=&quot;text&quot;
placeholder=&quot;Name&quot;
value={attendee.name}
onChange={(e) =&gt; {
const updated = [...attendees];
updated[index].name = e.target.value;
setAttendees(updated);
}}
/&gt;
&lt;input
type=&quot;email&quot;

placeholder=&quot;Email&quot;
value={attendee.email}
onChange={(e) =&gt; {
const updated = [...attendees];
updated[index].email = e.target.value;
setAttendees(updated);
}}
/&gt;
&lt;/div&gt;
))}

&lt;button
onClick={() =&gt; setAttendees([...attendees, { name: &#39;&#39;, email: &#39;&#39; }])}
&gt;
Add Attendee
&lt;/button&gt;

&lt;button onClick={handleScheduleSession}&gt;Schedule Session&lt;/button&gt;

&lt;h2&gt;Scheduled Sessions&lt;/h2&gt;
&lt;ul&gt;
{sessions.map((session) =&gt; (
&lt;li key={session._id}&gt;
{new Date(session.start).toLocaleString()} - {new Date(session.end).toLocaleString()} ({session.duration}
minutes)
&lt;ul&gt;
{session.attendees.map((attendee, idx) =&gt; (
&lt;li key={idx}&gt;{attendee.name} ({attendee.email})&lt;/li&gt;
))}
&lt;/ul&gt;
&lt;/li&gt;
))}
&lt;/ul&gt;

&lt;/div&gt;
);
};

export default AdminDashboard;
Q3.
Backend Enhancements:
1. Modify Session (Reschedule):
o PUT /api/schedule/:id allows rescheduling sessions.
2. Cancel Session:
o DELETE /api/schedule/:id deletes a session.
3. Notification Preferences:
o A user schema enhancement for notification preferences (email, sms).

// Modify session (Reschedule)
app.put(&#39;/api/schedule/:id&#39;, async (req, res) =&gt; {
try {
const { id } = req.params;
const { start, end } = req.body;

// Check for conflicts
const existingSession = await Session.find({
_id: { $ne: id },
$or: [
{ start: { $lt: new Date(end) }, end: { $gt: new Date(start) } },
],
});

if (existingSession.length &gt; 0) {
return res.status(400).send({ message: &#39;Conflict with existing sessions&#39; });
}

const updatedSession = await Session.findByIdAndUpdate(
id,

{ start, end },
{ new: true }
);

res.status(200).send(updatedSession);
// Add notification logic here
} catch (error) {
res.status(400).send(error);
}
});

// Cancel session
app.delete(&#39;/api/schedule/:id&#39;, async (req, res) =&gt; {
try {
const { id } = req.params;

const deletedSession = await Session.findByIdAndDelete(id);
res.status(200).send(deletedSession);

// Add notification logic here
} catch (error) {
res.status(400).send(error);
}
});

// User notification preferences
const userSchema = new mongoose.Schema({
email: { type: String, required: true },
notificationPreferences: {
email: { type: Boolean, default: true },
sms: { type: Boolean, default: false },
},
});

const User = mongoose.model(&#39;User&#39;, userSchema);
Q4.
1. HTML Structure for Scheduling and Availability
&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;
&lt;meta charset=&quot;UTF-8&quot;&gt;
&lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
&lt;title&gt;Scheduling and Availability&lt;/title&gt;
&lt;link rel=&quot;stylesheet&quot; href=&quot;styles.css&quot;&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;header&gt;
&lt;h1&gt;Manage Your Availability&lt;/h1&gt;
&lt;/header&gt;

&lt;section class=&quot;availability&quot;&gt;
&lt;h2&gt;Available Time Slots&lt;/h2&gt;
&lt;div class=&quot;time-slots&quot;&gt;
&lt;button class=&quot;slot&quot;&gt;9:00 AM&lt;/button&gt;
&lt;button class=&quot;slot&quot;&gt;10:00 AM&lt;/button&gt;
&lt;button class=&quot;slot&quot;&gt;11:00 AM&lt;/button&gt;
&lt;button class=&quot;slot&quot;&gt;1:00 PM&lt;/button&gt;
&lt;button class=&quot;slot&quot;&gt;2:00 PM&lt;/button&gt;
&lt;/div&gt;
&lt;/section&gt;

&lt;section class=&quot;sessions&quot;&gt;
&lt;h2&gt;Your Sessions&lt;/h2&gt;
&lt;ul class=&quot;session-list&quot;&gt;
&lt;li&gt;Session 1: 9:00 AM - 10:00 AM&lt;/li&gt;
&lt;li&gt;Session 2: 1:00 PM - 2:00 PM&lt;/li&gt;

&lt;/ul&gt;
&lt;button class=&quot;add-session-btn&quot;&gt;Add New Session&lt;/button&gt;
&lt;/section&gt;

&lt;footer&gt;
&lt;p&gt;© 2025 Your Company. All rights reserved.&lt;/p&gt;
&lt;/footer&gt;

&lt;script src=&quot;script.js&quot;&gt;&lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;
2. CSS for Responsive and Accessible Design
/* styles.css */
/* Global styles */
body {
font-family: Arial, sans-serif;
margin: 0;
padding: 0;
box-sizing: border-box;
background-color: #f4f4f9;
}
header, footer {
background-color: #282c34;
color: #ffffff;
text-align: center;
padding: 15px;
}
h1, h2 {
margin: 0;
padding: 0;
}
button {
padding: 10px;
margin: 5px;
font-size: 16px;
cursor: pointer;
border: none;
background-color: #4CAF50;
color: white;
border-radius: 4px;
}

button:hover {
background-color: #45a049;
}
button:disabled {
background-color: #c1c1c1;
cursor: not-allowed;
}
/* Time slots and session list */
.time-slots {
display: flex;
flex-wrap: wrap;
justify-content: center;
margin-top: 20px;
}
.slot {
width: 150px;
margin: 10px;
text-align: center;
}
/* Responsive Layout */
.sessions, .availability {
margin: 20px;
}
.session-list {
list-style-type: none;
padding: 0;
}
.session-list li {
margin-bottom: 10px;
}
@media (max-width: 768px) {
.time-slots {
flex-direction: column;
align-items: center;
}
.slot {
width: 100%;
margin: 5px 0;
}
}
/* Accessibility: Ensure good color contrast */
button, h1, h2 {
color: #fff;

}
footer p {
font-size: 14px;
}
/* Focus styles for accessibility */
button:focus, .slot:focus {
outline: 3px solid #FFD700;
}
3. JavaScript for Managing Sessions and Availability
// script.js
document.addEventListener(&#39;DOMContentLoaded&#39;, function () {
// Handle adding new session
const addSessionBtn = document.querySelector(&#39;.add-session-btn&#39;);
addSessionBtn.addEventListener(&#39;click&#39;, function () {
const availableSlots = document.querySelectorAll(&#39;.slot&#39;);
availableSlots.forEach(slot =&gt; {
slot.addEventListener(&#39;click&#39;, function () {
const sessionTime = this.textContent;
addSession(sessionTime);
});
});
});
// Function to add a session to the list
function addSession(time) {
const sessionList = document.querySelector(&#39;.session-list&#39;);
const sessionItem = document.createElement(&#39;li&#39;);
sessionItem.textContent = `Session: ${time}`;
sessionList.appendChild(sessionItem);
}
});

1. HTML
a. How do you ensure accessibility when creating a web page?
 Semantic HTML: Use proper HTML elements like &lt;header&gt;, &lt;footer&gt;, &lt;nav&gt;, &lt;article&gt;, and
&lt;section&gt; to help screen readers understand the page structure.
 ARIA (Accessible Rich Internet Applications): Use ARIA attributes like aria-label, aria-
describedby, and aria-hidden to improve accessibility.
 Color contrast: Ensure sufficient contrast between text and background for
readability.
 Keyboard accessibility: Ensure all interactive elements are focusable and usable with
keyboard navigation.
b. What are semantic HTML tags, and why are they important?

 Semantic HTML tags: These are HTML elements that carry meaning about their
content (e.g., &lt;header&gt;, &lt;footer&gt;, &lt;article&gt;, &lt;section&gt;, &lt;nav&gt;, etc.).
 Importance:
o They help with SEO.
o They improve accessibility for users with disabilities.
o They make the structure of the document clearer for developers and
maintainers.

c. How would you implement an HTML form with validation for email, phone number, and
password strength? Here’s an example of a form with validation:
html
Copy code
&lt;form id=&quot;userForm&quot;&gt;
&lt;label for=&quot;email&quot;&gt;Email:&lt;/label&gt;
&lt;input type=&quot;email&quot; id=&quot;email&quot; name=&quot;email&quot; required&gt;
&lt;label for=&quot;phone&quot;&gt;Phone:&lt;/label&gt;
&lt;input type=&quot;tel&quot; id=&quot;phone&quot; name=&quot;phone&quot; pattern=&quot;^[0-9]{10}$&quot; required&gt;
&lt;label for=&quot;password&quot;&gt;Password:&lt;/label&gt;
&lt;input type=&quot;password&quot; id=&quot;password&quot; name=&quot;password&quot; required minlength=&quot;8&quot; pattern=&quot;^(?=.*[A-Za-
z])(?=.*\d)[A-Za-z\d]{8,}$&quot;&gt;
&lt;button type=&quot;submit&quot;&gt;Submit&lt;/button&gt;
&lt;/form&gt;
&lt;script&gt;
document.getElementById(&#39;userForm&#39;).addEventListener(&#39;submit&#39;, function(e) {
const email = document.getElementById(&#39;email&#39;).value;
const phone = document.getElementById(&#39;phone&#39;).value;
const password = document.getElementById(&#39;password&#39;).value;
if (!email || !phone || !password) {
e.preventDefault();
alert(&quot;Please fill all fields correctly.&quot;);
}
});
&lt;/script&gt;

2. CSS
a. Explain the difference between relative, absolute, and fixed positioning in CSS.
 Relative positioning: An element is positioned relative to its normal position in the
document flow. You can adjust it using top, left, right, or bottom.
 Absolute positioning: An element is positioned relative to its nearest positioned
ancestor (i.e., one with relative, absolute, or fixed set). If no ancestor is found, it will be
relative to the &lt;html&gt; element.
 Fixed positioning: The element is fixed to the viewport, meaning it remains in the
same position even when the page is scrolled.

css
Copy code
.relative { position: relative; top: 20px; left: 30px; }
.absolute { position: absolute; top: 50px; left: 100px; }
.fixed { position: fixed; top: 10px; right: 10px; }
b. How would you create a responsive navigation bar without using any libraries?
Here’s a simple CSS-only responsive navigation bar:
html
Copy code
&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;
&lt;meta charset=&quot;UTF-8&quot;&gt;
&lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
&lt;title&gt;Responsive Navbar&lt;/title&gt;
&lt;style&gt;
body { font-family: Arial, sans-serif; }
nav { background-color: #333; }
nav ul { list-style-type: none; padding: 0; margin: 0; }
nav ul li { display: inline-block; }
nav ul li a { color: white; padding: 15px 20px; display: block; text-decoration: none; }
nav ul li a:hover { background-color: #ddd; color: black; }
/* Mobile Styles */
@media (max-width: 768px) {
nav ul { text-align: center; }
nav ul li { display: block; width: 100%; }
}
&lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;nav&gt;
&lt;ul&gt;
&lt;li&gt;&lt;a href=&quot;#home&quot;&gt;Home&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a href=&quot;#about&quot;&gt;About&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a href=&quot;#services&quot;&gt;Services&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a href=&quot;#contact&quot;&gt;Contact&lt;/a&gt;&lt;/li&gt;
&lt;/ul&gt;
&lt;/nav&gt;
&lt;/body&gt;
&lt;/html&gt;
c. Provide a solution for centering a div both horizontally and vertically using CSS.
You can use Flexbox to center a div both horizontally and vertically:
html
Copy code
&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;
&lt;meta charset=&quot;UTF-8&quot;&gt;

&lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
&lt;title&gt;Centered Div&lt;/title&gt;
&lt;style&gt;
.container {
display: flex;
justify-content: center;
align-items: center;
height: 100vh;
}
.box {
width: 200px;
height: 200px;
background-color: #4CAF50;
}
&lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;div class=&quot;container&quot;&gt;
&lt;div class=&quot;box&quot;&gt;&lt;/div&gt;
&lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;

3. JavaScript Problem-Solving
a. Write a function to check if a given string is a palindrome.
javascript
Copy code
function isPalindrome(str) {
const cleanedStr = str.replace(/[^a-zA-Z0-9]/g, &#39;&#39;).toLowerCase();
return cleanedStr === cleanedStr.split(&#39;&#39;).reverse().join(&#39;&#39;);
}
console.log(isPalindrome(&quot;A man, a plan, a canal, Panama&quot;)); // true
b. Implement a function to flatten a nested array.
javascript
Copy code
function flattenArray(arr) {
return arr.reduce((acc, val) =&gt; Array.isArray(val) ? acc.concat(flattenArray(val)) : acc.concat(val), []);
}
console.log(flattenArray([1, [2, [3, 4], 5], 6])); // [1, 2, 3, 4, 5, 6]
c. Implement a custom dropdown menu using JavaScript.
html
Copy code
&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;

&lt;meta charset=&quot;UTF-8&quot;&gt;
&lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
&lt;title&gt;Custom Dropdown&lt;/title&gt;
&lt;style&gt;
.dropdown { position: relative; display: inline-block; }
.dropdown-content { display: none; position: absolute; background-color: #f9f9f9; min-width: 160px; z-
index: 1; }
.dropdown:hover .dropdown-content { display: block; }
.dropdown-content a { color: black; padding: 12px 16px; text-decoration: none; display: block; }
.dropdown-content a:hover { background-color: #f1f1f1; }
&lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;div class=&quot;dropdown&quot;&gt;
&lt;button class=&quot;dropbtn&quot;&gt;Dropdown&lt;/button&gt;
&lt;div class=&quot;dropdown-content&quot;&gt;
&lt;a href=&quot;#&quot;&gt;Option 1&lt;/a&gt;
&lt;a href=&quot;#&quot;&gt;Option 2&lt;/a&gt;
&lt;a href=&quot;#&quot;&gt;Option 3&lt;/a&gt;
&lt;/div&gt;
&lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;

4. React
a. Explain the difference between controlled and uncontrolled components in React.
 Controlled Components: The form data is controlled by React state. The value of the
form element is set by the state, and updates are handled via onChange event.
 Uncontrolled Components: The form data is handled by the DOM. You access the
value of the form element via refs.
jsx
Copy code
// Controlled Component Example
function ControlledForm() {
const [email, setEmail] = useState(&quot;&quot;);
return (
&lt;input type=&quot;email&quot; value={email} onChange={(e) =&gt; setEmail(e.target.value)} /&gt;
);
}
b. How would you optimize a React app for performance?
 Use React.memo to prevent unnecessary re-renders of functional components.
 Use useCallback and useMemo to memoize functions and values.
 Implement code-splitting using React&#39;s React.lazy() and Suspense.
 Avoid inline functions inside JSX because they can cause unnecessary re-renders.

c. Implement a simple React component to display a paginated list of items.
jsx
Copy code
import React, { useState } from &#39;react&#39;;
const PaginatedList = ({ items }) =&gt; {
const [currentPage, setCurrentPage] = useState(1);
const itemsPerPage = 5;
const indexOfLastItem = currentPage * itemsPerPage;
const indexOfFirstItem = indexOfLastItem - itemsPerPage;
const currentItems = items.slice(index
