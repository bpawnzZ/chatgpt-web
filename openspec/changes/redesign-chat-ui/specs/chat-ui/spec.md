# Spec Delta

## Purpose

Defines the application shell and the conversation surface: how the window is divided between the chat rail and the conversation, how a message communicates who is speaking, and how the composer and theming behave — the part of the product a reader actually sits in.

## ADDED Requirements

### Requirement: No upstream author credit
The interface SHALL NOT display the upstream project's author credit, attribution link, or source-repository icon anywhere in the rendered application.

#### Scenario: Setup screen
- **WHEN** a reader opens the app root with no API key configured
- **THEN** no "by Niek van der Maas" credit, personal-site link, or GitHub mark is rendered

#### Scenario: Conversation view
- **WHEN** a reader is inside a chat
- **THEN** the area beneath the conversation and composer renders no attribution text or source icon

### Requirement: Messages carry speaker identity
Every rendered user and assistant message SHALL identify its speaker with a visible name, and assistant messages SHALL additionally show an avatar mark derived from the character name.

#### Scenario: Assistant message
- **WHEN** an assistant reply is rendered in a chat whose character name is "Aria"
- **THEN** the message shows an "A" avatar mark and the name "Aria" above the body text

#### Scenario: User message
- **WHEN** a user message is rendered
- **THEN** the message is visually distinguished from assistant messages and labelled as the reader's own turn

#### Scenario: Character name falls back
- **WHEN** the chat has no character name configured
- **THEN** assistant messages fall back to a default label rather than rendering an empty header

### Requirement: Chat rail collapses
The chat list rail SHALL be collapsible to zero width and restorable, so the conversation can occupy the full window.

#### Scenario: Collapse
- **WHEN** the reader toggles the rail closed
- **THEN** the rail leaves the layout, the conversation occupies the full window width, and a control to restore the rail remains reachable

#### Scenario: State persists
- **WHEN** the reader reloads the page after collapsing the rail
- **THEN** the rail is still collapsed

#### Scenario: Navigate with the rail open
- **WHEN** the reader selects a chat from the rail on a narrow viewport
- **THEN** the rail closes so the conversation is visible

### Requirement: Conversation is a bounded reading column
Conversation content SHALL be constrained to a comfortable reading measure and centred within the available width, rather than stretching to the viewport edge.

#### Scenario: Wide viewport
- **WHEN** the window is wider than the reading measure
- **THEN** message bodies stay within the measure and are horizontally centred in the content area

### Requirement: Composer stays reachable and grows with input
The message composer SHALL remain anchored at the bottom of the conversation while it scrolls, SHALL grow with multi-line input up to a limit, and SHALL submit on Enter with Shift+Enter inserting a newline.

#### Scenario: Long conversation
- **WHEN** the conversation is longer than the viewport and the reader scrolls
- **THEN** the composer stays visible and does not cover the last message

#### Scenario: Multi-line input
- **WHEN** the reader types several lines into the composer
- **THEN** the composer grows in height instead of scrolling internally

### Requirement: Coherent in light and dark
The interface SHALL render as a coherent design in both light and dark colour schemes, following the operating system preference without a separate in-app switch.

#### Scenario: System in dark mode
- **WHEN** the reader's system prefers a dark colour scheme
- **THEN** the shell, rail, messages, and composer use the dark palette and text remains legible against every surface

### Requirement: Presentation-only change
The redesign SHALL NOT alter stored chat or message data, the storage location of chats, messages, or API keys, or which endpoints requests are sent to.

#### Scenario: Existing chats after upgrade
- **WHEN** a reader with previously stored chats loads the redesigned interface
- **THEN** every stored chat, message, and setting is present and unchanged, and the API key is still read from browser storage only
