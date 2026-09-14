# TrustData Web Tag for Google Tag Manager

Send page views, events and consent decisions from a Google Tag Manager web
container to TrustData. The tag loads the TrustData SDK, reads Google Consent Mode,
and captures IAB TCF consent choices without extra setup.

Documentation: https://docs.trustdata.tech/

## Install the template

1. In Google Tag Manager, open **Templates**.
2. Under **Tag Templates**, click **Search Gallery**.
3. Search for **TrustData Web Tag** and click **Add to workspace**.

To install a version by hand, download `template.tpl` from this repository. In
**Templates**, click **New**, open the ⋮ menu, click **Import**, and select the file.

## Create the tag

1. In TrustData, open your property's **Attribution IDs** page.
2. Copy the Attribution ID of the web stream.
3. In Google Tag Manager, create a tag of type **TrustData Web Tag**.
4. Paste the Attribution ID.
5. Set **Command** to **Initialize + send event (All Pages)**.
6. Add the **All Pages** trigger and save.

Use the Attribution ID, not the property ID. Events sent with a property ID are not
attributed.

Most sites need only this one tag. Test it in **Preview** before you publish the
container.

## Commands

| Command | Trigger | What it does |
|---|---|---|
| Initialize + send event (All Pages) | All Pages | Loads the SDK and sends `page_view` |
| Send event | Any trigger | Sends an event. It loads the SDK first if no TrustData tag has |
| Send consent decision (CMP event) | Your CMP's consent event | Records a consent choice. Needed only when Decision capture is Manual |
| Initialize only | Consent Initialization or All Pages | Loads the SDK without sending an event |

Pick a standard event name such as `add_to_cart`, `begin_checkout` or `purchase`, or
choose **Custom…** for your own. Add event parameters in the table, or pass a variable
that returns an object, such as a Data Layer Variable for `ecommerce`. When both set
the same key, the table value is sent.

## Track funnel steps

Use a standard event name for a standard moment: `view_item`, `add_to_cart`,
`begin_checkout`, `add_payment_info`, `purchase`, `generate_lead`. TrustData places
these in a funnel with no setup, counts them in sessions, and forwards the ones ad
platforms support.

Use `funnel_step` for a step that has no standard name, such as passenger details or
cabin selection. Choose **funnel_step** as the event name and fill in two fields:

- **Step index**: the step's position in the funnel.
- **Step name**: a stable ID such as `passenger_details`. Keep it the same when the
  page changes, because funnels match on it.

If every step is a `funnel_step`, number them 1, 2, 3. If the funnel also uses
standard events, number around their positions: `view_item` 10, `add_to_cart` 20,
`begin_checkout` 30, `add_payment_info` 40, `purchase` 90. A step between checkout and
payment is 31, and the next one is 32.

A `funnel_step` tag without a step index or a step name fails and sends nothing. Add
`value` and `currency` in the event parameters to record the basket at that step.

## Consent

The tag reads Google Consent Mode by default: `analytics_storage`, `ad_storage`,
`functionality_storage` and `ad_user_data`. Without analytics consent, the SDK stores
nothing on the device and sends no visitor ID.

**Decision capture** controls how a visitor's choice is recorded:

- **Auto (IAB TCF)** is the default. The tag listens for the TCF user action event,
  so any TCF consent platform works with no extra tag.
- **Manual** is for consent platforms without TCF. Add a second tag with the command
  **Send consent decision (CMP event)** and fire it on your CMP's consent event.

If your consent platform does not set Consent Mode, set **Consent source** to
**Manual (CMP variables)** and map each consent type to a variable.

## Personal data

Pass an email or phone in `user_data`. The tag hashes both with SHA-256 in the
browser before it sends the event. A value that is already a SHA-256 hash is sent
unchanged.

- Emails are trimmed and lowercased before hashing.
- A phone number that starts with `+` is hashed as it is.
- A phone number in national format, such as `06 01 02 03 04`, needs **Default phone
  country code**. Without it, the tag drops the phone instead of hashing the wrong
  number.
- If hashing fails, the tag drops that field. It never sends an unhashed value.

The tag never writes an email or phone number to the console, including in Preview.

## Permissions

| Permission | Scope | Why |
|---|---|---|
| Injects scripts | `https://t.trustdata.tech/t.js` | Load the TrustData SDK |
| Accesses global variables | `trustdata` (read, write), `trustdata._loaded` (read) | Queue commands until the SDK loads |
| Accesses global variables | `trustdata.init`, `.event`, `.consent`, `.setUserId`, `.linkIdentity`, `.debug` (execute) | Call the loaded SDK |
| Accesses global variables | `__tcfapi` (read, execute) | Listen for TCF consent choices |
| Accesses consent state | `analytics_storage`, `ad_storage`, `functionality_storage`, `ad_user_data` (read) | Read Consent Mode and listen for changes |
| Logs to console | Debug mode only | Show what the tag does in Preview |

## Troubleshooting

**The tag shows Failed in Tag Assistant.** The browser did not load `t.js`. An ad
blocker or a Content Security Policy usually blocks it. Add
`https://t.trustdata.tech` to `script-src` and `connect-src` in your Content Security
Policy.

**The tag succeeds but no events arrive.** Check that the Attribution ID belongs to
the web stream of the property you are looking at in TrustData.

**Preview shows "Phone dropped".** The phone is in national format. Set **Default
phone country code**.

**A funnel_step tag fails.** Set **Step index** to 1 or more and fill in **Step name**.

## Support

Open an issue in this repository, or email contact@trustdata.tech.

## License

Apache License 2.0. See [LICENSE](LICENSE).
