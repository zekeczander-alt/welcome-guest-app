I'm looking to develop an app for small scale AirBnB owners. The first iteration of this app will cater to only one business, but the architecture will later be extended to host other AirBnB owners also to manage their property.

Host/Owner of the AirBnB, would be able to set up it's property landing page, featuring pictures, description and various amenities that the facility provides to their guests. The host is further onboard the staff members at the facility such as kitchen and housekeeping staff.

Anybody on the property's page would be able to sign in with their Google account and be able to view generic information published on the site.

Host is also able to define the various parts of the property that is rentable e.g. twin room, room-1, lawn area, tour packages etc.

To manage guests, the host is able to generate a reservation on the property. The reservation contains various fields such as duration, room numbers, number of adults and children in the party, special instructions for the guest's stay and additional offerings during their stay etc.

The host is also able to add profiles of each adult member of the party to the reservation. The profiles of the members can also be created by their Google account or whatsApp number.

Once the reservation is created, a presigned URL to access reservation is sent to user's email or whatsApp number. The URL contains a custom page which provides guests about important information about their stay such as below:
	- Personalised fields
		- Rooms reserved
		- Duration of stay
		- Link to view or download copy of the bill.
		- Link to communicate with host and staff over WhatsApp.
	- Generic fields
		- Facilities
		- Kitchen menu
		- Tour packages
		- Attractions around
		- Other public links for the property and reviews on those sites.

This presigned URL allows user to login without username and password, but also allows them to update their profile and set password, but the app allows access to the guest only for the duration of the stay.

Unauthenticated users for the property would see a static page with description and and generic fields mentioned above.

The host would be able to define values for the above fields.

The development of this app would be done using NextJS framework and other technologies popular with developers of this ecosystem.

Please generate a detailed Product Requirement Document based on the information provided above.