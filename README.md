# <img width="25" height="35" alt="Titled_Tuesday_allmode" src="https://github.com/user-attachments/assets/9862a504-61b3-4a75-b4da-75e02d3ef6e5" /> Titled Tuesday Chess Analytics
Dive into the fast-paced world of Titled Tuesday with a Power BI dashboard that transforms thousands of chess games into insightful stories about players, performance, strategies, and the dynamics of elite online chess on <img width="100" height="30" alt="Chess-com-Logo-Transparent" src="https://github.com/user-attachments/assets/36145a87-5a61-4f7c-be51-8499b2bb8724" />


## <img width="25" height="25" alt="image" src="https://github.com/user-attachments/assets/556883bb-703a-426c-9d0b-7e06cd0a8351" /> Objectives of this project

**Examining Performance Patterns:** An Analysis of the Relationship Between Game Accuracy and Player Performance Across Skill Levels in Chess.com Titled Tuesday Tournaments.

**Development of an end-to-end solution:** Demonstration of the full data workflow (ETL)—importing a raw dataset; transforming and splitting data into four relational tables using SQL; designing a data schema (star schema); and building an interactive dashboard in Power BI.

**A data-centric approach to chess:** identifying the most effective players, analyzing the impact of piece color on win rates, and detecting anomalies (paradoxes) over the course of long tournaments.

## <img width="25" height="25" alt="image" src="https://github.com/user-attachments/assets/4667705e-82a1-47db-bd9c-b70c1247afe4" /> Data overview

**Data volume:** The dashboard covers a vast dataset of over **34,000 matches** played (from September 26, 2023 to December 3, 2024).

**Time dimension:** Data are aggregated by tournament rounds (from round 1 to 11), allowing for the assessment of player dynamics over time.

**Key Performance Indicators (KPIs) displayed on the dashboard:**
* **Win Rate (White / Black):** Overall win percentage in the tournament (in the screenshot: 78.62% and 73.05% for the selected segments).

* **Average Accuracy:** The average accuracy of a player's moves according to Chess.com algorithms.

* **Brilliant Games (%)** — the share of games containing "brilliant" moves out of the total number of games.

* **Rating Tier:** Classification of players into rating categories, ranging from Candidate Masters (up to 2200) to Super Grandmasters (2700+).


<img width="818" height="456" alt="tykjytjrjjutrjty" src="https://github.com/user-attachments/assets/d8ca26a6-3f54-4ed0-a5fa-a5c5d0ba12b1" />


## <img width="25" height="25" alt="image" src="https://github.com/user-attachments/assets/a8b199ac-b5e2-4f50-a169-d89d6b2c181b" /> Tech Stack & Architecture

The project was implemented using the classic architecture for corporate analytics solutions (DWH/BI). All logic, aggregations, and complex statistical metrics are calculated at the **SQL** level, ensuring maximum performance and a lightweight final model in **Power BI**.

### <img width="20" height="20" alt="image" src="https://github.com/user-attachments/assets/e5bfbf8b-84e2-4433-9ab3-0ee11b536d72" />   SQL Stage: Transformation and Metric Calculation (ELT Layer)
  The raw flat dataset [titled_tuesday](./titled_tuesday.csv) was normalized and split into **4 clean analytical tables** using optimized SQL queries:

* **<ins>Side_Color_Analysis (Color Effectiveness Analysis):</ins>**

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> Use of CTEs (Common Table Expressions) and the analytic window function `SUM(...) OVER(PARTITION BY...)`.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Metrics:** Calculation of the win rate percentage `(win_rate_percentage)`, as well as the dynamic calculation of the player's accuracy delta relative to the global average `(accuracy_delta_from_global)` using a `CROSS JOIN`.

<details>
  <summary>📄 SQL Query (Click to expand)</summary>
  
  ```sql
-- Side_Color_Analysis
WITH color_stats AS (
    SELECT 
        CASE 
            WHEN rating BETWEEN 101 AND 2200 THEN '1. Candidate Master /CM / I Grade (до 2200)'
            WHEN rating BETWEEN 2201 AND 2500 THEN '2. FIDE Master / FM / IM (2201-2500)'
            WHEN rating BETWEEN 2501 AND 2700 THEN '3. Grandmaster Elite (2501-2700)'
            ELSE '4. Super Grandmasters / Top world (2700+)'
        END AS rating_tier,
        username,
        CASE WHEN white = 'True' THEN 'White' ELSE 'Black' END AS color_side,
        COUNT(*) AS total_games,
        AVG(accuracy) AS avg_accuracy,
        SUM(score) * 100.0 / COUNT(*) AS win_rate_percentage
    FROM titled_tuesday
    WHERE rating > 100 AND accuracy > 1.0 AND username IS NOT NULL AND score IS NOT NULL
    GROUP BY 1, 2, white
),
global_stats AS (
    SELECT AVG(accuracy) AS global_avg_accuracy FROM titled_tuesday WHERE rating > 100 AND accuracy > 1.0 AND username IS NOT NULL
)
SELECT 
    c.rating_tier,
    c.username,
    c.color_side,
    c.total_games,
    ROUND(c.avg_accuracy, 2) AS avg_accuracy,
    ROUND(c.avg_accuracy - g.global_avg_accuracy, 2) AS accuracy_delta_from_global,
    ROUND(c.win_rate_percentage, 2) AS win_rate_percentage,
    ROUND(c.total_games * 100.0 / SUM(c.total_games) OVER(PARTITION BY c.rating_tier, c.username), 2) AS games_share_pct
FROM color_stats c
CROSS JOIN global_stats g;
```

</details>




* **<ins>Round_Dynamics (Tournament dynamics by round):</ins>**

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> Use of the `LAG()` window shift function and the `SUM(...) OVER(ORDER BY...)` cumulative sum function.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Metrics:** Tracking accuracy progress compared to the previous round `(accuracy_growth_from_prev_round)` and a cumulative count of games played throughout the tournament `(cumulative_games_played)`.

<details>
  <summary>📄 SQL Query (Click to expand)</summary>
  
  ```sql
-- Round_Dynamics
SELECT 
    rating_tier,
    username,
    round,
    games_in_round,
    avg_accuracy_in_round,
    ROUND(avg_accuracy_in_round - LAG(avg_accuracy_in_round, 1) OVER (PARTITION BY rating_tier, username ORDER BY round), 2) AS accuracy_growth_from_prev_round,
    avg_score_in_round,
    SUM(games_in_round) OVER (PARTITION BY rating_tier, username ORDER BY round ASC) AS cumulative_games_played
FROM (
    SELECT 
        CASE 
            WHEN rating BETWEEN 101 AND 2200 THEN '1. Candidate Master /CM / I Grade (до 2200)'
            WHEN rating BETWEEN 2201 AND 2500 THEN '2. FIDE Master / FM / IM (2201-2500)'
            WHEN rating BETWEEN 2501 AND 2700 THEN '3. Grandmaster Elite (2501-2700)'
            ELSE '4. Super Grandmasters / Top world (2700+)'
        END AS rating_tier,
        username,
        round,
        COUNT(*) AS games_in_round,
        ROUND(AVG(accuracy), 2) AS avg_accuracy_in_round,
        ROUND(AVG(score), 3) AS avg_score_in_round
    FROM titled_tuesday
    WHERE rating > 100 AND accuracy > 1.0 AND username IS NOT NULL AND round IS NOT NULL
    GROUP BY 1, 2, round
) sub
ORDER BY rating_tier, username, round ASC;
```

</details>





* **<ins>Top_10_Elite (Ranking of the best players):</ins>**

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> Filtering the dataset using `HAVING COUNT(*) >= 100` (to ensure statistical validity) and applying `DENSE_RANK()` for ranking.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Metrics:** Mathematical calculation of the root mean square deviation (standard deviation \(\sigma \)) using the formula `SQRT(AVG(x²) - AVG(x)²)` to assess the stability and consistency of a chess player's move accuracy `(accuracy_consistency_sigma)`.

<details>
  <summary>📄 SQL Query (Click to expand)</summary>
  
  ```sql
-- Top_10_Elite
WITH ranked_players AS (
    SELECT 
        CASE 
            WHEN rating BETWEEN 101 AND 2200 THEN '1. Candidate Master /CM / I Grade (до 2200)'
            WHEN rating BETWEEN 2201 AND 2500 THEN '2. FIDE Master / FM / IM (2201-2500)'
            WHEN rating BETWEEN 2501 AND 2700 THEN '3. Grandmaster Elite (2501-2700)'
            ELSE '4. Super Grandmasters / Top world (2700+)'
        END AS rating_tier,
        username,
        COUNT(*) AS total_games_played,
        ROUND(AVG(rating), 0) AS avg_player_rating,
        ROUND(AVG(accuracy), 2) AS elite_avg_accuracy,
        ROUND(SQRT(AVG(accuracy * accuracy) - (AVG(accuracy) * AVG(accuracy))), 2) AS accuracy_consistency_sigma,
        SUM(score) AS total_points_scored,
        DENSE_RANK() OVER (PARTITION BY 
            CASE 
                WHEN rating BETWEEN 101 AND 2200 THEN '1. Candidate Master /CM / I Grade (до 2200)'
                WHEN rating BETWEEN 2201 AND 2500 THEN '2. FIDE Master / FM / IM (2201-2500)'
                WHEN rating BETWEEN 2501 AND 2700 THEN '3. Grandmaster Elite (2501-2700)'
                ELSE '4. Super Grandmasters / Top world (2700+)'
            END 
            ORDER BY AVG(accuracy) DESC) AS internal_rank
    FROM titled_tuesday
    WHERE rating > 100 AND accuracy > 1.0 AND username IS NOT NULL
    GROUP BY 1, 2
    HAVING COUNT(*) >= 100
)
SELECT * 
FROM ranked_players
WHERE internal_rank <= 10
ORDER BY rating_tier, internal_rank ASC;
```

</details>







* **<ins>Rating_Segmentation (Qualification-based segmentation):</ins>**

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> Complex conditional aggregation using `SUM(CASE WHEN...)` and grouping.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Metrics:** Determination of the number and share of "brilliant games" (`brilliant_games_count` / `brilliant_games_share_pct`), where player accuracy reached or exceeded the 95.0% threshold.

<details>
  <summary>📄 SQL Query (Click to expand)</summary>
  
  ```sql
-- Rating_Segmentation
SELECT 
    CASE 
        WHEN rating BETWEEN 101 AND 2200 THEN '1. Candidate Master /CM / I Grade (до 2200)'
        WHEN rating BETWEEN 2201 AND 2500 THEN '2. FIDE Master / FM / IM (2201-2500)'
        WHEN rating BETWEEN 2501 AND 2700 THEN '3. Grandmaster Elite (2501-2700)'
        ELSE '4. Super Grandmasters / Top world (2700+)'
    END AS rating_tier,
    username,
    COUNT(*) AS total_games_played,
    ROUND(AVG(accuracy), 2) AS avg_accuracy_in_tier,
    ROUND(AVG(score), 3) AS avg_score_in_tier,
    SUM(CASE WHEN accuracy >= 95.0 THEN 1 ELSE 0 END) AS brilliant_games_count,
    ROUND(SUM(CASE WHEN accuracy >= 95.0 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS brilliant_games_share_pct
FROM titled_tuesday
WHERE rating > 100 AND accuracy > 1.0 AND username IS NOT NULL
GROUP BY 1, 2
ORDER BY 1, 3 DESC;
```

</details>

### <img width="20" height="20" alt="image" src="https://github.com/user-attachments/assets/c5a2c748-fa81-4ada-8e9c-0d8a491786aa" /> Power BI: Modeling and UI (Presentation Layer)
By moving calculations to the database, the Power BI model became extremely lightweight and high-performing:

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Data schema:** Clear relational links were established between four imported tables using the `rating_tier` and `username` keys.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **DAX Optimization:** A separate, isolated helper table was generated using DAX to improve the visual, sorting, and customization of the user-facing filter (Slicer).

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **UI/UX:** Interactive cross-filtering has been implemented—selecting a rating tier or a specific chess player instantly updates the accuracy, round, and color preference charts.




## <img width="25" height="25" alt="image" src="https://github.com/user-attachments/assets/67dad89f-7be3-415c-b199-7bcaf3bc427c" />  **Insights & Analysis**

* ### Top 10 Players by Accuracy and Consistency

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **The clear leaders:** Players **magnuscarlsen** (90.38) and **dropstonedp** (90.33) are demonstrating the highest average accuracy in the tournament.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Elite-level density:** All top-10 players have an average accuracy ranging from **89.39% to 0.38%**. This highlights the incredibly slim margin for error at the highest level—a split second or a micro-mistake determines the winner.

* ### The Accuracy–Performance Paradox by Round

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Visualized trend:** The graph shows a clear anomaly. In the initial rounds (1–2), players' average accuracy is at its highest **(0.92 and 0.86)**, yet the average score (Average Score line) remains low.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **The essence of the paradox is this:** as the tournament progresses (rounds 8–11), average accuracy drops to **0.73–0.75**, whereas the Average Score rises sharply.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/625134af-f111-421e-a8ea-a4c0f2581fe2" /> **Chess context:** At the start of a tournament, elite players face weaker opponents, playing according to rigorous positional chess theory (characterized by high computer-like precision). In the final rounds, the leaders face one another: positions become extremely sharp, chaotic, and tactical. Players consciously take risks—situations where computer-calculated "precision" drops, but practical effectiveness and tension rise.

* ### Distribution of games by piece color

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Tournament balance:** The number of games played with the white pieces is **17,497 (50.97%)**, and with the black pieces — **16,830 (49.03%)**.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/625134af-f111-421e-a8ea-a4c0f2581fe2" /> **Analysis:** The tournament organizers ensure near-perfect mathematical parity in the distribution of colors, which eliminates the factor of the random first-move advantage (for White) over the long run for an individual player.

* ### Performance by Rating Tier

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Super Grandmaster Dominance:** The Super Grandmaster category (2700+) is the most representative, with **33,566 games** played and an average accuracy of **81.90%**. This is the core of the Titled Tuesday tournament.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> **Brilliant Games Anomaly:** The highest percentage of "brilliant games" **(Brilliant Games (%) = 24.24%)** was recorded in the FIDE Master / FM / IM category (2201–2500).

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/625134af-f111-421e-a8ea-a4c0f2581fe2" /> **Analysis:** Lower-rated players (2201–2500) are compelled to play extremely aggressive, unconventional, and sharp chess to pose challenges to opponents rated 2700+. This generates more moves that the Chess.com algorithm rates as "Brilliant." At the same time, super-grandmasters (2700+) maintain a high level of consistency (16.63% "Brilliant" moves across a vast number of games).

## Business Value & Recommendations
Based on the developed dashboard, the following data-driven solutions have been formulated for practical application in media production, digital marketing, and gaming platform gamification:

* ### Media Content & Live Streaming Monetization (Media & Streaming Value):

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> The discovered "Round Paradox" (where player accuracy and concentration peak during the final rounds) provides a clear, actionable recommendation for streaming platforms and commentators.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/625134af-f111-421e-a8ea-a4c0f2581fe2" /> Since the highest-rated opponents face each other in rounds 8 to 11, elevating the quality of chess to its absolute limits, this final third of the tournament attracts the highest viewer engagement. Marketers can leverage this insight to integrate premium ad placements and schedule high-value sponsorship integrations during peak broadcast viewership.

* ### Enhancing User Engagement (User Retention & Platform Gamification):

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> The anomaly discovered in the "Brilliant Games" share (`brilliant_games_share_pct`) within the 2201–2500 rating tier proves that mid-tier masters generate more unconventional, aggressive, and highly entertaining content.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/625134af-f111-421e-a8ea-a4c0f2581fe2" /> Platforms (such as Chess.com) can implement automated Premium subscription recommendation funnels by curating weekly "Top Tactical Highlights" specifically from this player segment, directly driving platform activity and engagement among casual users.

* ### Tournament Balance & Fairness Management (Tournament Data Management):

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/48c4ff08-2a9d-4d3c-9718-eee6f10e87a1" /> The proven mathematical equity in piece color distribution (approximately 51% to 49% for White / Black) validates the high quality of the pairings and matchmaking algorithms.

<img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/625134af-f111-421e-a8ea-a4c0f2581fe2" /> This allows organizers of major online esports events to guarantee absolute "Fair Play" and enhance the commercial appeal of their tournaments for corporate sponsors, as the random first-move advantage factor is completely neutralized over the long run.

### Thank you for your interest in this project <img width="35" height="35" alt="image" src="https://github.com/user-attachments/assets/a8686c00-0cb6-431a-aeac-f0a4cbc82a09" />












