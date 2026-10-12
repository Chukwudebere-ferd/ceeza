# ceeza
Complete booking meeting SYSTEM

Understood. We'll focus only on the MVP algorithms for your Zoho Bookings-style platform using Go, React + TypeScript, PostgreSQL, hosted on Zeabur. No code or infrastructure tutorial.
The key is to build one booking engine that handles availability, appointments, Google Meet links, calendar invitations and reminders.
1. Complete booking algorithm
Business configures services
Duration, price, staff, working hours, buffers and meeting type

Customer opens booking page
Selects service, staff member, date and available time

Availability engine
Calculates free slots from schedules, existing bookings, time zones and buffers

Booking transaction
Rechecks availability and reserves the slot without double booking

Confirmation and integrations
Google Meet link · Google Calendar event · email confirmation · reminders


2. Google Meet + Google Calendar: how they connect
The business owner connects their Google account once through Google OAuth. Your backend stores the authorization securely and uses it to create calendar events on the owner's behalf, with the permissions the owner granted. Google recommends requesting only the necessary OAuth scopes; public apps may need verification for certain scopes. 

Google for Developers
+1



The booking flow then works like this:
1. Create the booking in PostgreSQL first. Generate a unique booking ID and mark it confirmed or pending, according to your rules.
2. Create a Google Calendar event. Include the service, customer, start/end time, attendees and booking reference.
3. Request a Google Meet conference. In the same event request, use conferenceData.createRequest with a unique requestId and set conferenceDataVersion=1. Google generates the meeting details asynchronously. 
   
   Google for Developers
   +1
4. Save the integration result. Store the Google event ID and Meet URL against your booking. If Meet creation is still pending, check the event again until it succeeds or fails.
5. Send the customer confirmation. Include the date, time, time zone, booking reference, Meet link and calendar invitation/add-to-calendar option.
Important: Google Meet generation can fail or remain pending. Your booking must not be lost just because Google is temporarily unavailable. Use a retryable background job and show the Meet link as pending until it is ready.
3. “Add to Calendar” — support two paths
These are different features:
- Automatic Google Calendar integration: the business connects Google, and your system creates/updates/deletes events in that calendar.
- Customer's Add to Calendar button: the customer can add the appointment to their own calendar, even if they haven't connected Google to your platform.
For the MVP, offer Google Calendar and downloadable .ics invitations. If the business's Google integration is active, create the event there too. Google Calendar supports event creation and attendee invitations through its API. 

Google for Developers
+1



4. The algorithms your MVP needs
Algorithm	Rules
Availability	Business hours minus breaks, blocked dates, existing bookings and buffer periods
Booking	Recheck the selected slot transactionally before confirming
Google integration	Create event → request Meet link → save integration status
Rescheduling	Lock/check the new slot, update the booking and sync the existing calendar event
Cancellation	Cancel booking, release the slot, delete or cancel the calendar event and stop reminders
Reminders	Queue jobs at configured intervals; skip jobs for cancelled bookings
Retry and recovery	Retry temporary Google failures without creating duplicate events
Critical database rule: enforce overlapping-booking prevention at the database level for each staff member or other bookable resource. Checking availability only in the frontend is insufficient because two customers can select the same slot simultaneously.
5. Recommended MVP booking states
PENDING
Booking created; confirmation or integration still processing

CONFIRMED
Appointment is reserved

RESCHEDULED
Appointment time changed; integrations updated

CANCELLED
Slot released and pending reminders invalidated

COMPLETED
Appointment finished


Keep the booking status separate from integration statuses. A booking can be confirmed while Google Calendar synchronization is retrying.
My recommendation: implement the internal booking engine first, then Google OAuth and calendar events, then Meet generation, followed by rescheduling/cancellation synchronization and reminders. That gives you a usable booking product without making every booking dependent on Google being available.
