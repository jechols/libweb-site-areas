# Site Areas Module

## Overview

This module gives admins the ability to manage a taxonomy of site areas which
can be used for access control (locking a user/manager to their "area" of the
site) as well as determining things like menus and URL prefixes.  Users are
given access to one or more site areas, and, when creating or updating nodes,
cannot select a site area that isn't in their allowed areas list.

This module creates a "Content areas" taxonomy, a new "Allowed areas" field for
users, and a "magic" field for nodes to use to designate that they want this
module to apply its magic.

Any node that uses the area-specific field (see "setup" below) becomes
protected by this module.  Using that field signifies that you want this module
to apply its magic to the node based on the user's "Allowed areas" value(s)
(`field_allowed_areas`).

*If the two fields (users' allowed areas and a node's content area) aren't
referencing the same taxonomy, or aren't even taxonomy references to begin
with, behavior is undefined, but it probably won't be what you want.*

- On creation of a node that has a site area, the valid options are restricted
  to those in the user's allowed areas.
- When editing a node, the same restriction applies.
- No user can edit nodes set to an area outside their allowed areas, even if
  they created it originally.
- Area admins can be created to manage all content within their allowed areas,
  allowing for a safe choice that's higher than normal users, but lower than
  full content administrators.
- `content_areas` terms (like all Taxonomies) can be set up hierarchically.
  Users with access to a top-level area will automatically be granted access to
  all the children of that area.

## Permissions

"Area admins" can be created with the `administer any node in allowed areas`
permission.  This gives "area admins" the ability to edit or delete others'
content within their allowed area(s), while content in other areas is still off
limits.

`administer any node in allowed areas` doesn't magically give users node
permissions, so they still need to be given other permissions for editing
whatever nodes they need to be in charge of.  For instance, an area admin would
need permissions to do things like "edit any content", "delete any content",
etc. -- for whatever node types are used.

## "Sub-areas"

As mentioned previously, `content_areas` is a standard Drupal taxonomy, and as
such, its terms can have children and deeper descendants as needed. Only the
top-level element is listed when choosing users' allowed areas, but they are
given the same access to all descendants of any chosen areas.

For example, if "Borrowing" were a sub-area of "Access Services", a curator of
"Access Services" is implicitly able to curate "Borrowing" content.

## Setup

### Install

- Install "Site Areas" from the "UO Libraries" module section
- Add content areas (`/admin/structure/taxonomy/manage/content_areas/overview`)
  - Create a hierarchy as desired to reflect "sub-areas" of the site.
- Make the user field show up on the user form (**not** the display, just the form)
  - Visit `/admin/config/people/accounts/form-display`
  - Move "Allowed areas" out of the "Disabled" group
  - Save

### Content types

- To add the magic field to other content types:
  - Add a field (e.g., `/admin/structure/types/manage/CONTENT-TYPE/fields/add-field`)
    - Choose an existing field: `Entity reference: field_site_areas__site_area`
    - Change the label as desired and "Save and continue"
  - Settings:
    - Make the field required
    - Choose the "Content areas" Vocabulary
    - **Do not set a default for this field**
    - "Save settings"
  - Visit "Manage form display" to make "Site area" use "Check boxes / radio
    buttons"
  - Visit "Manage display" to make "Site area" hidden since it's just an
    administrative field
  - This field can now be used in rules for this content type's URL alias /
    menu / etc. - but nothing magic happens out of the box
- The "Basic Page" content type will automatically get the "Site area" field
  added to it.  This should probably be customized:
  - Visit `/admin/structure/types/manage/page/fields`
  - Remove the field from here if desired, otherwise...
  - Visit "Manage form display" and enable site areas by moving it out of the
    "disabled" section
  - While still on "Manage form display", make "Site area" use "Check boxes /
    radio buttons"
  - Visit "Manage display" and make sure the field is in the "Disabled" section

### Users

- Area-restricted curators:
  - Set up allowed areas on their user profile to restrict where they can
    create new content
    - Remember, if the `content_areas` taxonomy has a hierarchy, you only
      choose top-level elements, and the curator is automatically given access
      to all descendants.
  - Give them a role with permission to create content
    - Make sure the content types they can create have the site area field or
      are content types you are okay with being "site-wide"
- Area admins:
  - Set up allowed areas on their user profile to restrict where they can
    administer content
    - Same as curators: area admins are automatically given access to all
      chosen areas' descendants.
  - Give them a role with permissions to fully manage (create, edit, delete,
    manage revisions, etc.) content
    - Make sure the content types they manage have the site area field or
      are content types you are okay with them administering site-wide
    - They will need permissions like "edit any content", "delete any content",
      etc., not just creating and editing own content
  - Make sure area admins are also given a role which has the "administer any
    node in allowed areas" permission, otherwise the other roles won't actually
    grant anything on area-protected content types

## Usage

After all setup has completed, all nodes with the "Site area" field are
protected to a degree.  When creating nodes that use this field, a user will
only be able to select site areas that are in their allowed list (or a child of
areas in their allowed list).  Users will be unable to edit or delete content
outside their area.  And "area managers" will be content admins limited to
whatever is in their allowed areas list.

**Note, however,** that nodes lacking this field will not be affected in any
way by this module: users' allowed areas have no affect and this module's
permissions have no effect.  For example, if site admins are given access to
"Article: edit any content", and the Article type doesn't use this field, they
will have unrestricted access.  Their list of allowed areas will have no
meaning in that context.
