graph TD
  %% Client
  subgraph Client
    UI["React SPA"]
    Pages["Home, NewsFeed, ArticleDetail, Quizzes, QuizRunner, Results, Leaderboard"]
    FE_Services["Front-end services\nnewsApi, authService"]
  end

  %% Firebase
  subgraph Firebase
    Hosting["Firebase Hosting\nserves frontend/build"]
    Auth["Firebase Auth"]
    Firestore["Cloud Firestore"]
    CF_API["Cloud Functions\napi onRequest (Express)"]
    CF_Sched["Cloud Functions\nscheduled jobs"]
  end

  %% Server services
  subgraph Server_Services["Functions services layer"]
    ArticleGen["articleGenerator.js\ngenerateBalancedArticle\nmodel: Perplexity sonar"]
    QuizSvc["quizService.js\ngenerateQuizQuestions + generateQuizSummary\nmodel: Perplexity sonar-pro"]
    MarketSvc["marketDataService.js\nFinancial Modeling Prep"]
  end

  %% External
  subgraph External
    PPLX_SONAR["Perplexity API\nchat/completions\nmodel: sonar"]
    PPLX_SONAR_PRO["Perplexity API\nchat/completions\nmodel: sonar-pro"]
    FMP["Financial Modeling Prep API"]
    Picsum["Picsum placeholder images"]
  end

  %% Delivery and auth
  UI -->|served| Hosting
  UI -->|sign in| Auth

  %% Client to API
  FE_Services -->|/api/articles, /api/quizzes, /api/leaderboard, /api/market-data| CF_API

  %% Articles flow
  CF_API -->|POST /api/articles/generate| ArticleGen
  ArticleGen --> PPLX_SONAR
  ArticleGen --> Firestore
  UI -->|GET /api/articles| CF_API
  CF_API --> Firestore

  %% Quizzes flow
  CF_API -->|POST /api/quizzes/generate| QuizSvc
  QuizSvc --> PPLX_SONAR_PRO
  QuizSvc --> Firestore
  UI -->|POST /api/quizzes/submit| CF_API
  CF_API --> QuizSvc
  QuizSvc --> PPLX_SONAR_PRO
  CF_API --> Firestore

  %% Leaderboards and market
  CF_API --> MarketSvc
  MarketSvc --> FMP
  MarketSvc --> Firestore

  %% Schedules
  CF_Sched -->|hourly| MarketSvc
  CF_Sched -->|daily weekly monthly resets| Firestore
