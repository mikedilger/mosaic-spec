# Contacts

The contact list is a list of pubkeys that a user considers legitimate
(neither spam nor malicious). They are not necessarily friends, and they
are not determiniate of a user's feed. Clients should automatically add
keys to this list when a user behaves in a way that indicates the key is
not spam or malicious, such as when a user replies to one of their posts.

The details of this are TBD.

## Web of Trust

The purpose of the list is to create a web of trust among shared contacts,
that can be used as a way to detect and filter spam and malicious actors.

## Nicknames

Additionally, users can nickname other users if the other user's 'name'
collides amongst their contacts, or if they just find it easier to remember
them by a different nickname.
