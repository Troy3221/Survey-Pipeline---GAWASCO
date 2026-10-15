This single notebook merges the two GAWASCO notebooks end-to-end, in the order the work is actually meant to happen:

    Build the clean database — append this run's raw export, classify every no-account-number row as Potential Customer or Reject, remove duplicate account numbers, match against the company account database, save the clean workbook, then correct it for wrong-GPS submissions (records whose GPS point landed on the road instead of the customer's premise get moved from All Data (Clean) into Rejects), and finally add a Zone Progress sheet — a live coverage tracker by zone (total/visited/not-visited/coverage %, colour-scaled, with a bar chart) — right into the workbook. This step produces the one "true" clean database for the run.
    Progress report — a Word summary of coverage, this run's processing stats, zone-wise coverage (table + a colour-coded coverage-by-zone chart), and daily enumerator performance, built from the corrected clean database.
    Remaining accounts — a PDF of accounts not yet visited (zone → route), also built from the corrected clean database.
    Weekly submissions report — a supporting Excel breakdown of accepted/rejected submissions per collector, per day, for a chosen date range.
    Day-specific extras — an Enumerator Submission Summary sheet, a Reject List sheet, and a narrative Word analysis report, all filtered to one specific date (REPORT_DATE) and added onto the same corrected workbook.

Why this order matters: the not-visited PDF and the progress report now read the database after GPS-correction, so an account whose only submission got GPS-rejected correctly shows up as "not yet visited" / a rejection, rather than being counted as clean before the correction ever ran.

One consequence of reordering: since Steps 2–5 all read the clean database back from disk after it's been saved and GPS-corrected, "today's" accepted/rejected counts (used in the progress report and the Enumerator Summary sheet) are computed by filtering the saved workbook on REPORT_DATE, rather than an in-memory tag. Both places that need "today's" counts now share one function (build_daily_counts), so the numbers can't drift apart.

Dates: every date column (submission time and interview date) is displayed as dd/mm/yyyy throughout the output workbooks. For compiling a day's enumerator output — the daily counts, the Reject List sheet, and the Word analysis report — submission time is the primary date; interview date is only used as a fallback if submission time is blank. Set REPORT_DATE to the actual date RAW_FILE was submitted on so "today's" counts line up.

How to use this each run:

    Update the Configuration cell (file paths, dates, REPORT_DATE, REPORT_DATE_LABEL(_FULL)).
    Run All. If you don't have a wrong-GPS file for this run, set INCLUDE_GPS_CORRECTION_STAGE = False — the pipeline still runs end-to-end, just skips the correction.

Potential vs Reject rule: a row with no account number is a Potential Customer if Q1. Is GAWASCO Service available at this property? is No. Any other no-account row is a Reject. This rule is re-applied to the entire dataset each run.

Handling form changes (Q-number drift): raw columns are reconciled to the master schema by question text, ignoring the Q<number>. prefix, so form renumbering doesn't silently lose or scramble data.
