When user provides the youtube link and ask for the context in this link

Step 1: Accept a YouTube Link and Validate
Input: User provides a YouTube video URL (e.g., https://www.youtube.com/watch?v=xyz123).
Validation:
Confirm the link is valid and follows YouTube's standard format.
Check if the video's transcript is officially available (not auto-generated subtitles).
Step 2: Retrieve Transcription via kome.ai
Use the tool: Visit https://kome.ai/tools/youtube-transcript-generator
Steps:
Paste the YouTube link into the input field.
Click "Generate Transcript".
Click "Copy" to Copy the full transcript text (including timestamps like [00:12:30]).
Step 3: Format the Transcript
Convert raw transcript into a structured Markdown format for readability:

[00:00 - 00:30] | Introduction  

- Speaker: "Welcome to today’s video where we’ll discuss..."  
- [Note]: Opening lacks clear context for beginner audiences.  

[00:30 - 05:00] | Technical Explanation  

- Speaker: "Profit = Revenue - Costs..."  
- [Clarification]: "Costs" should specify fixed vs. variable costs.  
Rules:
If timestamps are missing, label as [Unclear timing].
Highlight incomplete sections (e.g., pauses, unclear audio).
Use bold labels (e.g., [Note], [Clarification]) for analysis.
Step 4: Analyze for Weaknesses & Propose Improvements
Identified Issues
Problem Type	Examples
Missing content	Gaps between timestamps (e.g., [05:00] → [05:10] skipped).
Inconsistent timing	Timestamp overlaps or illogical time jumps.
Ambiguous language	Jargon without explanation (e.g., "ROI" without definition).
Factual inaccuracies	Incorrect data (e.g., "Discounts generate higher profits always").
Weak logic flow	Conclusion contradicts earlier discussion.

Step 5: Save to transcript.md



Preferred output language for textual content

When generating textual content or essays

When generating textual content or essays, the user prefers the output to be in Vietnamese.



Diagram Draw preference for essays

When composing essays or technical documents that can benefit from visual aids

Always draw diagrams in essays to help users easily understand the content.



User's preferred working language

When interacting with the user

The user prefers to use Vietnamese as the working language for all interactions.



Document Export Preferences

When exporting or generating documents.

When exporting documents, use the `adoc` format with a clear table of contents (`toc`), section numbering (`sectnums`), and ensure that each reference in the bibliography is on a new line



Information Search Preferences

When searching for information or retrieving data.

When searching for information, prioritize English websites, official and authoritative sources. Avoid information from personal blogs, unverified websites (e.g., Medium, Wikipedia, Reddit).



Document Reference Requirement

When generating documents.

Always include a 'References' section with clear citations for easy verification.

.
