# Spec Delta

## Purpose

Defines how a chat acquires its display name: the default seed every new chat starts with, the one-time model-generated name applied on the first exchange, and the guarantee that a name the reader chose is never overwritten.

## ADDED Requirements

### Requirement: New chats start with a numeric seed name
A newly created chat SHALL have a placeholder name of the form `Chat <number>` that is unique among existing chats.

#### Scenario: Creating a chat
- **WHEN** the reader starts a new chat while an earlier chat is named "Chat 1"
- **THEN** the new chat's name is a distinct seed name and no existing chat is renamed

### Requirement: First exchange auto-names the chat
After the first assistant reply completes in a chat that still carries its seed name, the app SHALL request a short topic summary from the configured model and use it as the chat name.

#### Scenario: First reply arrives
- **WHEN** a chat named "Chat 3" receives its first assistant reply
- **THEN** a naming request is sent to the same configured provider, and on success the chat's name becomes a short topic label and the chat list shows it without a reload

#### Scenario: Naming request fails
- **WHEN** the naming request errors, times out, or returns an unusable response
- **THEN** the chat keeps its seed name, no error message is inserted into the conversation, and the naming request is not retried automatically

### Requirement: Auto-naming does not touch the conversation
The naming request SHALL NOT add, alter, or delete messages in the chat, SHALL NOT be included in the token totals shown in the conversation, and SHALL NOT block the composer.

#### Scenario: Transcript unchanged
- **WHEN** auto-naming completes after the first exchange
- **THEN** the conversation contains exactly the messages that were sent and received, and the reader can send the next message while naming is still in flight

### Requirement: A reader-chosen name is permanent
Once the reader renames a chat, by the sidebar editor, the header rename action, or any other naming control, that chat SHALL NOT be auto-named later.

#### Scenario: Reader renamed before first reply completes
- **WHEN** the reader renames a chat before the automatic naming would apply
- **THEN** the chat keeps the reader's name and no automatic naming request is sent

#### Scenario: Reader renames after auto-naming
- **WHEN** the reader renames an auto-named chat
- **THEN** the new name is stored and never replaced

### Requirement: Pre-existing chats are not retroactively renamed
Chats that already contain messages when this behaviour is introduced SHALL keep their current names; auto-naming applies only to a chat's first exchange from that point forward.

#### Scenario: Older chat reopened
- **WHEN** a reader reopens a chat that already had several exchanges before the upgrade
- **THEN** its name is unchanged and no naming request is made for it
