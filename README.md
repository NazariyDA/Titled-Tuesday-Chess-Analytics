# <img width="25" height="35" alt="Titled_Tuesday_allmode" src="https://github.com/user-attachments/assets/9862a504-61b3-4a75-b4da-75e02d3ef6e5" /> Titled Tuesday Chess Analytics
Dive into the fast-paced world of Titled Tuesday with a Power BI dashboard that transforms thousands of chess games into insightful stories about players, performance, strategies, and the dynamics of elite online chess on <img width="100" height="30" alt="Chess-com-Logo-Transparent" src="https://github.com/user-attachments/assets/36145a87-5a61-4f7c-be51-8499b2bb8724" />

:pushpin: Data source

🧩 Used technologies: Power BI with SQL (data cleaning and transformation)

## <img width="25" height="25" alt="image" src="https://github.com/user-attachments/assets/190d7aa4-6ac1-4060-bf2b-7c58be120a0c" /> Objectives of this project

**Examining Performance Patterns:** An Analysis of the Relationship Between Game Accuracy and Player Performance Across Skill Levels in Chess.com Titled Tuesday Tournaments.

**Development of an end-to-end solution:** Demonstration of the full data workflow (ETL)—importing a raw dataset; transforming and splitting data into four relational tables using SQL; designing a data schema (star schema); and building an interactive dashboard in Power BI.

**A data-centric approach to chess:** identifying the most effective players, analyzing the impact of piece color on win rates, and detecting anomalies (paradoxes) over the course of long tournaments.

## <img width="25" height="25" alt="image" src="https://github.com/user-attachments/assets/2e00bec0-d629-4368-9657-90e933350f74" /> Data overview

**Data volume:** The dashboard covers a vast dataset of over 34,000 matches played (from September 26, 2023 to December 3, 2024).

**Time dimension:** Data are aggregated by tournament rounds (from round 1 to 11), allowing for the assessment of player dynamics over time.

**Key Performance Indicators (KPIs) displayed on the dashboard:**
* **Win Rate (White / Black):** Overall win percentage in the tournament (in the screenshot: 78.62% and 73.05% for the selected segments).

* **Average Accuracy:** The average accuracy of a player's moves according to Chess.com algorithms.

* **Brilliant Games (%)** — the share of games containing "brilliant" moves out of the total number of games.

* **Rating Tier:** Classification of players into rating categories, ranging from Candidate Masters (up to 2200) to Super Grandmasters (2700+).

## <img width="25" height="25" alt="image" src="https://github.com/user-attachments/assets/6793dbc4-3d2d-4088-bf15-df94174bfd8f" /> **Insights & Analysis**

* ### Top 10 Players by Accuracy and Consistency

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **The clear leaders:** Players magnuscarlsen (90.38) and dropstonedp (90.33) are demonstrating the highest average accuracy in the tournament.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/55601706-b5a9-4da3-9c7a-8926a6ee60b2" /> **Elite-level density:** All top-10 players have an average accuracy ranging from 89.39% to 90.38%. This highlights the incredibly slim margin for error at the highest level—a split second or a micro-mistake determines the winner.

* ### The Accuracy–Performance Paradox by Round

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Visualized trend:** The graph shows a clear anomaly. In the initial rounds (1–2), players' average accuracy is at its highest (0.92 and 0.86), yet the average score (Average Score line) remains low.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **The essence of the paradox is this:** as the tournament progresses (rounds 8–11), average accuracy drops to 0.73–0.75, whereas the Average Score rises sharply.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Chess context:** At the start of a tournament, elite players face weaker opponents, playing according to rigorous positional chess theory (characterized by high computer-like precision). In the final rounds, the leaders face one another: positions become extremely sharp, chaotic, and tactical. Players consciously take risks—situations where computer-calculated "precision" drops, but practical effectiveness and tension rise.

* ### Distribution of games by piece color

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Tournament balance:** The number of games played with the white pieces is 17,497 (50.97%), and with the black pieces — 16,830 (49.03%).

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Analysis:** The tournament organizers ensure near-perfect mathematical parity in the distribution of colors, which eliminates the factor of the random first-move advantage (for White) over the long run for an individual player.

* ### Performance by Rating Tier

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Super Grandmaster Dominance:** The Super Grandmaster category (2700+) is the most representative, with 33,566 games played and an average accuracy of 81.90%. This is the core of the Titled Tuesday tournament.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Brilliant Games Anomaly:** The highest percentage of "brilliant games" (Brilliant Games (%) = 24.24%) was recorded in the FIDE Master / FM / IM category (2201–2500).

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Analysis:** Lower-rated players (2201–2500) are compelled to play extremely aggressive, unconventional, and sharp chess to pose challenges to opponents rated 2700+. This generates more moves that the Chess.com algorithm rates as "Brilliant." At the same time, super-grandmasters (2700+) maintain a high level of consistency (16.63% "Brilliant" moves across a vast number of games).














