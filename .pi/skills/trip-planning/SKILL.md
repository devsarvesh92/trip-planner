---
name: trip-planning
description: Plan and maintain shareable family trip itineraries with live booking research. Use when checking hotel availability (letsbook.me / ipms247 booking engines), searching India domestic flights, generating a shareable HTML travel plan with booking links and Google Maps links, tracking budgets, or versioning trip plans in git/GitHub.
---

# Trip Planning

Workflow for researching, budgeting, and publishing a shareable trip plan for this project.

## Project layout

- `index.html` — shareable, printable travel plan page (self-contained, no build step)
- `trip-plan.json` — structured source data (hotel, flights, maps, budget)
- `README.md` — links and GitHub Pages publishing notes

Keep `index.html` and `trip-plan.json` in sync when dates, prices, or bookings change.

## Hotel availability research (letsbook.me / ipms247 engine)

letsbook.me booking pages are backed by the `commonservice.ipms247.com` API. To query availability programmatically:

1. Fetch hotel details to get `hotelCode` and the auth token:
   ```
   GET https://commonservice.ipms247.com/YCSAPIServices/booking/gethoteldetails?propertySlug=<slug>
   ```
   The response `data._xk` is the auth token **reversed** — use `Authorization: Bearer <_xk reversed>` for subsequent calls.
2. Availability for a stay:
   ```
   GET .../booking/getAvailability?hotelCode=<code>&checkinDate=YYYY-MM-DD&checkoutDate=YYYY-MM-DD&adults=2&child=1&rooms=1&refresh=false&languageCode=en&childAge=<age>
   ```
   Results include `availableRooms`, `maxChildren`, `ratePlanMinNights`, and per-night `pricePerNight` (`discountedPrice`, `totalTaxes`, `priceAfterTax`).
3. Daily availability calendar (7-day windows):
   ```
   GET .../booking/availabilitycalendar?hotelCode=<code>&checkinDate=YYYY-MM-DD&adults=2&child=1&childAge=<age>
   ```
   Fetch 5 windows to cover a full month; find contiguous spans ≥ the rate plan's minimum nights.

Caveats:
- A room category may show nightly availability but report `maxChildren: 0` — always confirm child policy with the hotel directly before booking.
- `getAvailability` is the source of truth for a full stay; the calendar alone can overstate bookability.

## Flight research (India domestic)

Cleartrip's search API works without auth:

```
GET https://www.cleartrip.com/flight/search/v2?adults=2&childs=1&depart_date=DD/MM/YYYY&from=<IATA>&to=<IATA>&return_date=DD/MM/YYYY&class=Economy&trip_type=RoundTrip
```

Notes:
- The child passenger parameter is `childs` (not `children`).
- Response: `cards.J1` (outbound) and `cards.J2` (return); each card's `summary` has departure/arrival airports and times, stops, airlines (`metaData.airlineDetail`), and `lowestRoundTripPriceDetails.totalPricing.displayPrice` (round-trip total on outbound cards).
- Watch `nearbyAirports` on cards — a search for NMI may surface BOM departures; filter for exact origin/destination codes.
- Always cross-check final price and baggage on the airline's own site before recommending.

Useful airport codes for this project: `NMI` (Navi Mumbai Intl), `BOM` (Mumbai), `GOI` (Goa Dabolim — closest to Palolem/South Goa), `GOX` (Goa Mopa — North Goa).

## Google Maps link patterns

- Place: `https://www.google.com/maps/search/?api=1&query=<urlencoded name/address>`
- Directions: `https://www.google.com/maps/dir/?api=1&origin=<A>&destination=<B>`

## Publishing

- The plan is plain HTML/CSS/JS — preview with `open index.html`.
- Version with git; publish via GitHub Pages (Settings → Pages → deploy from `main` root).

## Reporting style

- Lead with the concrete recommendation (dates, room, flight numbers, times, totals in INR).
- Flag clearly when availability shows 0 rooms or when data comes from a third-party cache and must be re-verified.
- Include one clear "before you pay" checklist (exact room view confirmation, child policy, cancellation policy, final price with taxes).
