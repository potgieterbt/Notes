Created: December 7, 2022 9:47 PM
Updated: December 7, 2022 10:04 PM

A CLI app to help with studying and revision.

My study method is to once 8 have read something I will make a mock question and when revising I will try to answer the question then mark it based on a set answer or a googled response.

The app will help with:

1. First, the app will help randomise the questions, I will give a number as the total questions and the app will create a list of numbers ranging from 1 to the total questions and randomly select a number from the list and output it. It will then wait for a response(at first just pressing enter) then give the next number. The app will never repeate questions so once a question is answered that number will be removed from the list.
2. Next, the app will wait for a response for whether the answer to the question was correct or incorrect. This will then be put into a pandas df and later added to an ongoing csv file. I will have to input the subject and the section to keep csvs separate.
3. Last, I may add the function of being able to read the csv file of the section I input and it can calculate the recent correctness and using spaced repetition it can skip a question for a session or a period of time.
4. If this works well I may make it into a webapp to make it user friendly and I can create questions and have tests and have reports, spaced repetition, etc. I can use this for other things than just studying, I can add a large list of things I want to keep track of and do tests based on this.