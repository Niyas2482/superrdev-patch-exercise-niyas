# Patch Exercise Notes

## 1. What I fixed

I reviewed the application and focused on issues that could affect the correctness of the application and the user experience.

### Fix 1 - Status filter was not working correctly

File:
`backend/src/main/java/com/internal/tasktracker/TaskRepository.java`

The SQL query had an `AND` and `OR` condition without proper brackets. Because of SQL operator precedence, the status and archived conditions were not being applied correctly to both the title and description search.

I changed the query to group the title and description conditions together:

`AND (title LIKE ... OR description LIKE ...)`

and then applied the archived and status conditions.

I tested the API with the `OPEN` status and confirmed that the returned tasks had the correct status and were not archived.

### Fix 2 - Artificial delay in task search

File:
`backend/src/main/java/com/internal/tasktracker/TaskController.java`

There was a `Thread.sleep()` based on the search query length. This was adding an artificial delay to API requests, especially when the search text was empty or short.

I removed this delay because the API should not intentionally block the request based on the search text.

I tested the application again and confirmed that the API still returned the expected task data.

### Fix 3 - Pagination was not reset after filtering

File:
`frontend/src/App.jsx`

If the user was on a later page and then changed the search text or status filter, the application could continue using the old page number.

For example, if the user was on page 2 and selected a filter that only had one page of results, the application could request page 2 even though the filtered results started from page 1.

I changed the search and status filter handlers to reset the page to `1`.

I tested this in the UI by going to page 2 and then changing the search/status filter. The application correctly returned to page 1.

### Fix 4 - Loading state was not cleared when API request failed

File:
`frontend/src/hooks/useTasks.js`

When the API request failed, the `catch` block only stored the error message. It did not change `loading` back to `false`.

This could leave the UI in a loading state after an error.

I changed the error handling so that:
- `loading` is set to `true` when a request starts.
- Previous errors are cleared when a new request starts.
- `loading` is set to `false` when the request fails.

I also tested that the frontend still builds successfully.

## 2. What I did not change

I did not make large changes to the application because the exercise asks for a focused patch.

The backend currently gets all matching records and performs pagination in memory. I did not change this because moving pagination into the database would require a larger change to the repository/API implementation.

I also did not add request cancellation/debounce, input validation, or structured logging because these were lower priority compared with the correctness and blocking issues I fixed.

## 3. Biggest remaining risk

The biggest remaining risk is search performance when the database becomes large.

The current implementation loads all matching records before selecting the requested page. The search also uses `LOWER()` and `%searchTerm%`, which can become expensive for a large number of records.

A future improvement would be to implement database-level pagination and improve the search query/indexing strategy.

## 4. Verification

I verified the changes using:

- Backend Maven tests: `.\mvnw.cmd test`
- Frontend production build: `npm run build`
- API testing for status and search filters
- Browser testing for pagination/filter behaviour
- `git diff --check`

Backend tests completed successfully, and the frontend production build completed successfully.

## 5. Tools and AI usage

Tools used:
- VS Code
- PowerShell
- Maven
- npm/Vite
- Browser
- Git

I used ChatGPT as an assistant to understand the existing code, identify possible issues, understand the SQL condition problem, plan the fixes, and review the changes.

I made and reviewed the actual code changes myself and verified the application using API checks, browser testing, and build commands.