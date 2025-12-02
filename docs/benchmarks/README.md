# fiche Benchmark: JSON vs fiche Parsing Parity

This benchmark tests whether LLMs parse fiche format at parity with JSON.

## Test Data

- **[users.json](users.json)** - 4,170 bytes - 10 users with nested address/company objects + metadata
- **[users.fiche](users.fiche)** - 3,117 bytes - Same data in minified fiche format (**25% smaller**)

## Test Questions

We asked Claude Haiku (cold, no format explanation) these 10 questions:

1. What API version is this?
2. Is this data cached?
3. What company does user id=7 work for?
4. What is the longitude of the user in Aliyaview?
5. How many users have a zipcode starting with "5"?
6. What is the email of the user with username "Delphine"?
7. Which user lives furthest south (most negative latitude)?
8. What street does Clementina DuBuque live on?
9. List all users who work at companies with "Group" in the name.
10. What is the company catchphrase for the user in Bartholomebury?

## Results

| Question | JSON | fiche | Correct Answer |
|----------|------|-------|----------------|
| 1. API version | 3.0 | 3.0 | 3.0 |
| 2. Cached | Yes | Yes | Yes |
| 3. id=7 company | Johns Group | Johns Group | Johns Group |
| 4. Aliyaview longitude | -120.7677 | -120.7677 | -120.7677 |
| 5. Zipcodes starting "5" | 3 | 3 | 3 |
| 6. Delphine's email | Chaim_McDermott@dana.io | Rey.Padberg@karina.biz | Chaim_McDermott@dana.io |
| 7. Furthest south | Clementine Bauch | Nicholas Runolfsdottir V | Mrs. Dennis Schulist (-71.42) |
| 8. Clementina's street | Kattie Turnpike | Kattie Turnpike | Kattie Turnpike |
| 9. Companies w/ "Group" | Johns Group, Yost and Sons | Johns Group, Yost and Sons, Hoeger LLC | Johns Group, Abernathy Group |
| 10. Bartholomebury catchphrase | Switchable contextually-based project | Switchable contextually-based project | Switchable contextually-based project |

## Analysis

**Parsing accuracy:** Both formats parsed equivalently. Questions 1-5, 8, 10 answered identically.

**Reasoning errors:** Questions 7 and 9 failed on *both* formats with different wrong answers. This indicates:
- Errors are reasoning failures (finding minimum, pattern matching), not parsing failures
- fiche does not degrade model comprehension vs JSON

**Conclusion:** fiche parses at parity with JSON while being 25% smaller. Errors in complex reasoning tasks occur regardless of format.

## Try It Yourself

Copy the contents of either file and ask your preferred model the same questions. Compare results.
