# 🎮 Educational Games System - Complete Developer Documentation

## 📋 Table of Contents
1. [System Overview](#system-overview)
2. [Current Architecture Analysis](#current-architecture-analysis)
3. [Database Schema](#database-schema)
4. [Existing Implementation Issues](#existing-implementation-issues)
5. [Modern React/FastAPI Implementation](#modern-implementation)
6. [Installation & Setup Guide](#installation-setup)
7. [Troubleshooting Common Issues](#troubleshooting)

---

## 1. System Overview

### Technology Stack (Current)
- **Backend**: PHP 7.4+ with CodeIgniter v3
- **Database**: MySQL 8.0+ with InnoDB engine
- **Frontend**: Server-side rendered views with Bootstrap 4/5
- **Authentication**: RBAC (Role-Based Access Control)
- **Architecture**: Traditional MVC pattern

### Key Features
- **Dual Interface System**: Staff/Admin and Student portals
- **Game Types**: Quiz, Math, Language, Memory games
- **Real-time Leaderboards**: Class, School, and Subject rankings
- **Analytics Dashboard**: Performance metrics and progress tracking
- **Permission System**: Granular access control
- **Session Management**: Multi-attempt game sessions with time tracking

---

## 2. Current Architecture Analysis

### 2.1 File Structure
```
/app/
├── application/
│   ├── controllers/
│   │   ├── Games.php                    # Simple game controller
│   │   ├── admin/Gamebuilder.php        # Advanced admin controller
│   │   └── user/Gamebuilder.php         # Student controller
│   ├── models/
│   │   ├── Games_model.php              # Basic game model
│   │   └── Gamebuilder_model.php        # Advanced game model
│   └── views/
│       ├── games/                       # Simple views
│       └── gamebuilder/                 # Advanced views
├── backend/                             # Static assets
└── uploads/                             # File uploads
```

### 2.2 Controller Architecture

#### Games.php (Basic Implementation)
- **Purpose**: Simple game management
- **Methods**: admin(), create(), edit(), delete(), play(), submit_game()
- **Issues**: Basic authentication, limited features

#### admin/Gamebuilder.php (Advanced Implementation)
- **Purpose**: Full-featured admin interface
- **RBAC Integration**: Proper permission checking
- **Methods**: 
  - Game Management: index(), create(), edit(), delete()
  - Question Management: questions(), add_question()
  - Analytics: results(), dashboard(), leaderboard()
  - Student Interface: student_games(), play()

#### user/Gamebuilder.php (Student Interface)
- **Purpose**: Student game portal
- **Methods**: student_games(), play(), AJAX endpoints

### 2.3 Model Architecture

#### Games_model.php (Basic)
- Simple CRUD operations
- Basic leaderboard functionality
- Limited analytics

#### Gamebuilder_model.php (Advanced)
- **Comprehensive Features**:
  - Advanced game management with UUID
  - Multi-type question support
  - Session-based game play
  - Automated leaderboard updates
  - Statistical analytics
  - Permission-aware data filtering

### 2.4 Database Triggers & Automation
```sql
-- Automatic leaderboard updates
CREATE TRIGGER update_leaderboard_after_game_completion
-- Game statistics calculation
CREATE TRIGGER update_game_statistics
```

---

## 3. Database Schema

### 3.1 Core Tables

#### game_categories
```sql
CREATE TABLE `game_categories` (
  `id` int(11) PRIMARY KEY AUTO_INCREMENT,
  `name` varchar(100) NOT NULL,
  `description` text,
  `icon` varchar(50) DEFAULT 'fa-gamepad',
  `is_active` tinyint(1) DEFAULT 1,
  `created_at` timestamp DEFAULT CURRENT_TIMESTAMP
);
```

#### educational_games
```sql
CREATE TABLE `educational_games` (
  `id` int(11) PRIMARY KEY AUTO_INCREMENT,
  `uuid` char(36) NOT NULL UNIQUE,
  `title` varchar(255) NOT NULL,
  `description` text,
  `category_id` int(11) NOT NULL,
  `game_type` enum('quiz','math','language','memory') DEFAULT 'quiz',
  `subject_id` int(11) DEFAULT NULL,
  `class_id` int(11) DEFAULT NULL,
  `section_id` int(11) DEFAULT NULL,
  `created_by` int(11) NOT NULL,
  `difficulty_level` enum('easy','medium','hard') DEFAULT 'medium',
  `time_limit` int(11) DEFAULT NULL,
  `max_attempts` int(11) DEFAULT 3,
  `points_per_question` int(11) DEFAULT 10,
  `instructions` text,
  `is_active` tinyint(1) DEFAULT 1,
  `is_published` tinyint(1) DEFAULT 0,
  `created_at` timestamp DEFAULT CURRENT_TIMESTAMP
);
```

#### game_questions
```sql
CREATE TABLE `game_questions` (
  `id` int(11) PRIMARY KEY AUTO_INCREMENT,
  `game_id` int(11) NOT NULL,
  `question_text` text NOT NULL,
  `question_type` enum('multiple_choice','true_false','fill_blank','match_pairs') DEFAULT 'multiple_choice',
  `correct_answer` text NOT NULL,
  `points` int(11) DEFAULT 10,
  `explanation` text,
  `sort_order` int(11) DEFAULT 0,
  FOREIGN KEY (`game_id`) REFERENCES `educational_games`(`id`) ON DELETE CASCADE
);
```

#### game_sessions
```sql
CREATE TABLE `game_sessions` (
  `id` int(11) PRIMARY KEY AUTO_INCREMENT,
  `uuid` char(36) NOT NULL UNIQUE,
  `game_id` int(11) NOT NULL,
  `student_id` int(11) NOT NULL,
  `session_id` int(11) NOT NULL,
  `status` enum('in_progress','completed','abandoned','timeout') DEFAULT 'in_progress',
  `total_questions` int(11) DEFAULT 0,
  `correct_answers` int(11) DEFAULT 0,
  `total_score` int(11) DEFAULT 0,
  `time_taken` int(11) DEFAULT NULL,
  `attempt_number` int(11) DEFAULT 1,
  `started_at` timestamp DEFAULT CURRENT_TIMESTAMP,
  `completed_at` timestamp NULL DEFAULT NULL
);
```

#### game_leaderboards
```sql
CREATE TABLE `game_leaderboards` (
  `id` int(11) PRIMARY KEY AUTO_INCREMENT,
  `game_id` int(11) NOT NULL,
  `student_id` int(11) NOT NULL,
  `best_score` int(11) NOT NULL DEFAULT 0,
  `best_time` int(11) DEFAULT NULL,
  `total_attempts` int(11) DEFAULT 1,
  `class_rank` int(11) DEFAULT NULL,
  `school_rank` int(11) DEFAULT NULL,
  `subject_rank` int(11) DEFAULT NULL,
  `last_played` timestamp DEFAULT CURRENT_TIMESTAMP
);
```

### 3.2 Permission Tables Integration

#### RBAC Permission Setup
```sql
-- Staff permissions
INSERT INTO permission_group (name, short_code) VALUES ('Game Builder', 'game_builder');
INSERT INTO permission_category (perm_group_id, name, short_code, enable_view, enable_add, enable_edit, enable_delete) VALUES
(game_builder_group_id, 'Games Management', 'games_management', 1, 1, 1, 1),
(game_builder_group_id, 'Game Results', 'game_results', 1, 0, 0, 0);

-- Student module permissions
INSERT INTO permission_student (name, short_code, student, parent) VALUES 
('Educational Games', 'student_games', 1, 0);
```

---

## 4. Existing Implementation Issues

### 4.1 Common Problems You're Experiencing

#### Issue 1: Gaming Buttons Not Showing Up
**Root Cause**: Permission system not properly configured

**Diagnosis Steps**:
```sql
-- Check if Educational Games permissions exist
SELECT * FROM permission_category WHERE short_code LIKE '%game%';

-- Check staff role permissions
SELECT rp.*, pc.name as permission_name 
FROM roles_permissions rp 
JOIN permission_category pc ON pc.id = rp.perm_cat_id 
WHERE pc.short_code LIKE '%game%';

-- Check student module activation
SELECT * FROM permission_student WHERE short_code = 'student_games';
```

#### Issue 2: Buttons Not Working
**Root Cause**: Controller routing or permission middleware issues

**Check Route Configuration**:
```php
// In routes.php or .htaccess
$route['games/admin'] = 'games/admin';
$route['gamebuilder'] = 'admin/gamebuilder/index';
$route['user/games'] = 'user/gamebuilder/student_games';
```

#### Issue 3: Redirecting to Student Login
**Root Cause**: Authentication middleware incorrectly identifying user roles

**Debug Authentication**:
```php
// Check user data in session
$user_data = $this->customlib->getUserData();
echo '<pre>'; print_r($user_data); echo '</pre>';

// Verify role checking
if ($user_data['role'] != 'student') {
    // This condition might be failing
}
```

### 4.2 Permission System Fixes

#### Fix 1: Ensure Proper RBAC Setup
```sql
-- Grant permissions to Super Admin (role_id = 7)
INSERT IGNORE INTO roles_permissions (role_id, perm_cat_id, can_view, can_add, can_edit, can_delete)
SELECT 7, pc.id, 1, 1, 1, 1 
FROM permission_category pc 
WHERE pc.short_code IN ('games_management', 'game_results', 'game_analytics');

-- Grant permissions to Teachers (role_id = 2)
INSERT IGNORE INTO roles_permissions (role_id, perm_cat_id, can_view, can_add, can_edit, can_delete)
SELECT 2, pc.id, 1, 1, 1, 0 
FROM permission_category pc 
WHERE pc.short_code = 'games_management';
```

#### Fix 2: Enable Student Module
```sql
-- Enable Educational Games for students
UPDATE permission_student 
SET is_active = 1, student = 1 
WHERE short_code = 'student_games';
```

### 4.3 Menu Integration Issues

#### Staff Menu Configuration
The menu should be in: `/application/views/layout/sidebar.php`

```php
<!-- Check if user has game management permission -->
<?php if ($this->rbac->hasPrivilege('games_management', 'can_view')) { ?>
<li class="treeview">
    <a href="#">
        <i class="fa fa-gamepad"></i>
        <span>Educational Games</span>
        <span class="pull-right-container">
            <i class="fa fa-angle-left pull-right"></i>
        </span>
    </a>
    <ul class="treeview-menu">
        <li><a href="<?php echo base_url('gamebuilder'); ?>"><i class="fa fa-circle-o"></i> Manage Games</a></li>
        <li><a href="<?php echo base_url('gamebuilder/create'); ?>"><i class="fa fa-circle-o"></i> Create Game</a></li>
        <li><a href="<?php echo base_url('gamebuilder/results'); ?>"><i class="fa fa-circle-o"></i> View Results</a></li>
    </ul>
</li>
<?php } ?>
```

#### Student Menu Configuration
Student menu location: `/application/views/layout/student/header.php`

```php
<?php if ($this->module_lib->hasActive('student_games')) { ?>
<li>
    <a href="<?php echo base_url('user/gamebuilder/student_games'); ?>">
        <i class="fa fa-gamepad ftlayer"></i> 
        <span>Educational Games</span>
    </a>
</li>
<?php } ?>
```

---

## 5. Modern React/FastAPI Implementation

### 5.1 Technology Stack (Modern)
- **Backend**: FastAPI (Python 3.9+)
- **Database**: MongoDB with Mongoose ODM
- **Frontend**: React 18+ with TypeScript
- **State Management**: Redux Toolkit
- **UI Framework**: Tailwind CSS + Headless UI
- **Authentication**: JWT with role-based permissions
- **Real-time**: WebSocket for live leaderboards

### 5.2 Project Structure (Modern)
```
/app/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   ├── routes/
│   │   │   │   ├── auth.py
│   │   │   │   ├── games.py
│   │   │   │   ├── questions.py
│   │   │   │   └── leaderboard.py
│   │   │   └── dependencies.py
│   │   ├── models/
│   │   │   ├── game.py
│   │   │   ├── user.py
│   │   │   └── session.py
│   │   ├── services/
│   │   └── utils/
│   ├── requirements.txt
│   └── server.py
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── games/
│   │   │   ├── leaderboard/
│   │   │   └── dashboard/
│   │   ├── pages/
│   │   ├── store/
│   │   ├── services/
│   │   └── types/
│   ├── package.json
│   └── tailwind.config.js
└── README.md
```

### 5.3 Backend Implementation (FastAPI)

#### 5.3.1 Database Models
```python
# backend/app/models/game.py
from pydantic import BaseModel, Field
from typing import List, Optional, Literal
from datetime import datetime
from bson import ObjectId

class GameModel(BaseModel):
    id: Optional[str] = Field(alias="_id")
    title: str
    description: Optional[str] = None
    category: Literal["quiz", "math", "language", "memory"]
    subject_id: Optional[str] = None
    class_id: Optional[str] = None
    created_by: str
    difficulty: Literal["easy", "medium", "hard"] = "medium"
    time_limit: Optional[int] = None  # in minutes
    max_attempts: int = 3
    points_per_question: int = 10
    instructions: Optional[str] = None
    is_active: bool = True
    is_published: bool = False
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

class QuestionModel(BaseModel):
    id: Optional[str] = Field(alias="_id")
    game_id: str
    question_text: str
    question_type: Literal["multiple_choice", "true_false", "fill_blank", "match_pairs"]
    options: List[dict] = []  # For multiple choice
    correct_answer: str
    points: int = 10
    explanation: Optional[str] = None
    sort_order: int = 0

class GameSessionModel(BaseModel):
    id: Optional[str] = Field(alias="_id")
    game_id: str
    student_id: str
    status: Literal["in_progress", "completed", "abandoned", "timeout"] = "in_progress"
    total_questions: int
    answered_questions: int = 0
    correct_answers: int = 0
    total_score: int = 0
    time_taken: Optional[int] = None  # in seconds
    attempt_number: int = 1
    started_at: datetime = Field(default_factory=datetime.utcnow)
    completed_at: Optional[datetime] = None

class LeaderboardEntry(BaseModel):
    id: Optional[str] = Field(alias="_id")
    game_id: str
    student_id: str
    student_name: str
    student_photo: Optional[str] = None
    class_name: str
    best_score: int
    best_time: Optional[int] = None
    total_attempts: int
    class_rank: Optional[int] = None
    school_rank: Optional[int] = None
    subject_rank: Optional[int] = None
    last_played: datetime
```

#### 5.3.2 API Routes
```python
# backend/app/api/routes/games.py
from fastapi import APIRouter, Depends, HTTPException, status
from typing import List, Optional
from ..dependencies import get_current_user, require_permission
from ...models.game import GameModel, QuestionModel
from ...services.game_service import GameService

router = APIRouter(prefix="/api/games", tags=["games"])

@router.get("/", response_model=List[GameModel])
async def get_games(
    skip: int = 0,
    limit: int = 20,
    filter_by: Optional[str] = None,
    current_user: dict = Depends(get_current_user)
):
    """Get all games with pagination and filtering"""
    service = GameService()
    return await service.get_games(skip=skip, limit=limit, filter_by=filter_by, user=current_user)

@router.post("/", response_model=GameModel)
async def create_game(
    game: GameModel,
    current_user: dict = Depends(require_permission("games_management", "can_add"))
):
    """Create a new game"""
    service = GameService()
    game.created_by = current_user["id"]
    return await service.create_game(game)

@router.get("/{game_id}", response_model=GameModel)
async def get_game(
    game_id: str,
    current_user: dict = Depends(get_current_user)
):
    """Get single game by ID"""
    service = GameService()
    game = await service.get_game(game_id)
    if not game:
        raise HTTPException(status_code=404, detail="Game not found")
    return game

@router.put("/{game_id}", response_model=GameModel)
async def update_game(
    game_id: str,
    game_update: dict,
    current_user: dict = Depends(require_permission("games_management", "can_edit"))
):
    """Update existing game"""
    service = GameService()
    updated_game = await service.update_game(game_id, game_update)
    if not updated_game:
        raise HTTPException(status_code=404, detail="Game not found")
    return updated_game

@router.delete("/{game_id}")
async def delete_game(
    game_id: str,
    current_user: dict = Depends(require_permission("games_management", "can_delete"))
):
    """Delete game"""
    service = GameService()
    success = await service.delete_game(game_id)
    if not success:
        raise HTTPException(status_code=404, detail="Game not found")
    return {"message": "Game deleted successfully"}

# Question Management
@router.post("/{game_id}/questions", response_model=QuestionModel)
async def add_question(
    game_id: str,
    question: QuestionModel,
    current_user: dict = Depends(require_permission("games_management", "can_add"))
):
    """Add question to game"""
    service = GameService()
    question.game_id = game_id
    return await service.add_question(question)

@router.get("/{game_id}/questions", response_model=List[QuestionModel])
async def get_game_questions(
    game_id: str,
    current_user: dict = Depends(get_current_user)
):
    """Get all questions for a game"""
    service = GameService()
    return await service.get_game_questions(game_id)
```

#### 5.3.3 Game Service
```python
# backend/app/services/game_service.py
from motor.motor_asyncio import AsyncIOMotorDatabase
from typing import List, Optional, Dict
from ..models.game import GameModel, QuestionModel, GameSessionModel
from bson import ObjectId
from datetime import datetime

class GameService:
    def __init__(self, db: AsyncIOMotorDatabase):
        self.db = db
        self.games_collection = db.educational_games
        self.questions_collection = db.game_questions
        self.sessions_collection = db.game_sessions
        self.leaderboard_collection = db.game_leaderboards

    async def get_games(
        self, 
        skip: int = 0, 
        limit: int = 20, 
        filter_by: Optional[str] = None,
        user: dict = None
    ) -> List[GameModel]:
        """Get games with filtering and pagination"""
        query = {}
        
        # Apply user-based filtering for teachers
        if user and user["role"] == "teacher":
            query["created_by"] = user["id"]
        
        # Apply category filtering
        if filter_by:
            query["category"] = filter_by
        
        cursor = self.games_collection.find(query).skip(skip).limit(limit)
        games = await cursor.to_list(length=limit)
        
        # Add question count and player count
        for game in games:
            game["question_count"] = await self.questions_collection.count_documents(
                {"game_id": str(game["_id"])}
            )
            game["player_count"] = await self.sessions_collection.distinct(
                "student_id", {"game_id": str(game["_id"])}
            )
            game["player_count"] = len(game["player_count"])
        
        return [GameModel(**game) for game in games]

    async def create_game(self, game: GameModel) -> GameModel:
        """Create new game"""
        game_dict = game.dict(exclude={"id"})
        game_dict["created_at"] = datetime.utcnow()
        game_dict["updated_at"] = datetime.utcnow()
        
        result = await self.games_collection.insert_one(game_dict)
        game_dict["_id"] = result.inserted_id
        
        return GameModel(**game_dict)

    async def update_game(self, game_id: str, update_data: dict) -> Optional[GameModel]:
        """Update existing game"""
        update_data["updated_at"] = datetime.utcnow()
        
        result = await self.games_collection.update_one(
            {"_id": ObjectId(game_id)},
            {"$set": update_data}
        )
        
        if result.modified_count:
            updated_game = await self.games_collection.find_one({"_id": ObjectId(game_id)})
            return GameModel(**updated_game)
        
        return None

    async def delete_game(self, game_id: str) -> bool:
        """Delete game and all related data"""
        # Delete in correct order due to relationships
        await self.questions_collection.delete_many({"game_id": game_id})
        await self.sessions_collection.delete_many({"game_id": game_id})
        await self.leaderboard_collection.delete_many({"game_id": game_id})
        
        result = await self.games_collection.delete_one({"_id": ObjectId(game_id)})
        return result.deleted_count > 0

    async def start_game_session(self, game_id: str, student_id: str) -> Optional[str]:
        """Start new game session for student"""
        # Check if student has reached max attempts
        game = await self.games_collection.find_one({"_id": ObjectId(game_id)})
        if not game:
            return None
        
        attempt_count = await self.sessions_collection.count_documents({
            "game_id": game_id,
            "student_id": student_id
        })
        
        if attempt_count >= game["max_attempts"]:
            raise ValueError("Maximum attempts reached")
        
        # Get total questions
        total_questions = await self.questions_collection.count_documents(
            {"game_id": game_id}
        )
        
        session_data = {
            "game_id": game_id,
            "student_id": student_id,
            "status": "in_progress",
            "total_questions": total_questions,
            "answered_questions": 0,
            "correct_answers": 0,
            "total_score": 0,
            "attempt_number": attempt_count + 1,
            "started_at": datetime.utcnow()
        }
        
        result = await self.sessions_collection.insert_one(session_data)
        return str(result.inserted_id)

    async def submit_answer(
        self, 
        session_id: str, 
        question_id: str, 
        answer: str, 
        time_taken: Optional[int] = None
    ) -> bool:
        """Submit answer for a question"""
        # Get question details
        question = await self.questions_collection.find_one({"_id": ObjectId(question_id)})
        if not question:
            return False
        
        # Check if answer is correct
        is_correct = self._check_answer(question, answer)
        points = question["points"] if is_correct else 0
        
        # Update session statistics
        update_data = {
            "$inc": {
                "answered_questions": 1,
                "total_score": points
            }
        }
        
        if is_correct:
            update_data["$inc"]["correct_answers"] = 1
        
        await self.sessions_collection.update_one(
            {"_id": ObjectId(session_id)},
            update_data
        )
        
        return True

    async def complete_game_session(self, session_id: str, total_time: int) -> bool:
        """Complete game session"""
        update_data = {
            "status": "completed",
            "completed_at": datetime.utcnow(),
            "time_taken": total_time
        }
        
        result = await self.sessions_collection.update_one(
            {"_id": ObjectId(session_id)},
            {"$set": update_data}
        )
        
        if result.modified_count:
            # Update leaderboard
            await self._update_leaderboard(session_id)
            return True
        
        return False

    async def _update_leaderboard(self, session_id: str):
        """Update leaderboard after game completion"""
        session = await self.sessions_collection.find_one({"_id": ObjectId(session_id)})
        if not session:
            return
        
        # Update or create leaderboard entry
        existing_entry = await self.leaderboard_collection.find_one({
            "game_id": session["game_id"],
            "student_id": session["student_id"]
        })
        
        if existing_entry:
            # Update if this is a better score
            if session["total_score"] > existing_entry["best_score"]:
                await self.leaderboard_collection.update_one(
                    {"_id": existing_entry["_id"]},
                    {
                        "$set": {
                            "best_score": session["total_score"],
                            "best_time": session["time_taken"],
                            "last_played": session["completed_at"]
                        },
                        "$inc": {"total_attempts": 1}
                    }
                )
        else:
            # Create new entry
            leaderboard_entry = {
                "game_id": session["game_id"],
                "student_id": session["student_id"],
                "best_score": session["total_score"],
                "best_time": session["time_taken"],
                "total_attempts": 1,
                "last_played": session["completed_at"]
            }
            await self.leaderboard_collection.insert_one(leaderboard_entry)
        
        # Update rankings
        await self._calculate_rankings(session["game_id"])

    def _check_answer(self, question: dict, answer: str) -> bool:
        """Check if submitted answer is correct"""
        question_type = question["question_type"]
        correct_answer = question["correct_answer"]
        
        if question_type == "multiple_choice":
            return answer == correct_answer
        elif question_type == "true_false":
            return answer.lower() == correct_answer.lower()
        elif question_type == "fill_blank":
            return answer.lower().strip() == correct_answer.lower().strip()
        
        return False
```

### 5.4 Frontend Implementation (React + TypeScript)

#### 5.4.1 Types and Interfaces
```typescript
// frontend/src/types/game.ts
export interface Game {
  id: string;
  title: string;
  description?: string;
  category: 'quiz' | 'math' | 'language' | 'memory';
  subject_id?: string;
  class_id?: string;
  created_by: string;
  difficulty: 'easy' | 'medium' | 'hard';
  time_limit?: number;
  max_attempts: number;
  points_per_question: number;
  instructions?: string;
  is_active: boolean;
  is_published: boolean;
  created_at: string;
  updated_at: string;
  question_count?: number;
  player_count?: number;
}

export interface Question {
  id: string;
  game_id: string;
  question_text: string;
  question_type: 'multiple_choice' | 'true_false' | 'fill_blank' | 'match_pairs';
  options: QuestionOption[];
  correct_answer: string;
  points: number;
  explanation?: string;
  sort_order: number;
}

export interface QuestionOption {
  id: string;
  text: string;
  is_correct: boolean;
}

export interface GameSession {
  id: string;
  game_id: string;
  student_id: string;
  status: 'in_progress' | 'completed' | 'abandoned' | 'timeout';
  total_questions: number;
  answered_questions: number;
  correct_answers: number;
  total_score: number;
  time_taken?: number;
  attempt_number: number;
  started_at: string;
  completed_at?: string;
}

export interface LeaderboardEntry {
  id: string;
  game_id: string;
  student_id: string;
  student_name: string;
  student_photo?: string;
  class_name: string;
  best_score: number;
  best_time?: number;
  total_attempts: number;
  class_rank?: number;
  school_rank?: number;
  subject_rank?: number;
  last_played: string;
}
```

#### 5.4.2 Redux Store Setup
```typescript
// frontend/src/store/slices/gameSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { Game, Question, GameSession } from '../../types/game';
import { gameAPI } from '../../services/gameAPI';

interface GameState {
  games: Game[];
  currentGame: Game | null;
  questions: Question[];
  currentSession: GameSession | null;
  loading: boolean;
  error: string | null;
}

const initialState: GameState = {
  games: [],
  currentGame: null,
  questions: [],
  currentSession: null,
  loading: false,
  error: null,
};

// Async thunks
export const fetchGames = createAsyncThunk(
  'games/fetchGames',
  async (params: { skip?: number; limit?: number; filter_by?: string }) => {
    const response = await gameAPI.getGames(params);
    return response.data;
  }
);

export const createGame = createAsyncThunk(
  'games/createGame',
  async (gameData: Partial<Game>) => {
    const response = await gameAPI.createGame(gameData);
    return response.data;
  }
);

export const startGameSession = createAsyncThunk(
  'games/startSession',
  async (gameId: string) => {
    const response = await gameAPI.startGameSession(gameId);
    return response.data;
  }
);

const gameSlice = createSlice({
  name: 'games',
  initialState,
  reducers: {
    setCurrentGame: (state, action: PayloadAction<Game>) => {
      state.currentGame = action.payload;
    },
    clearCurrentGame: (state) => {
      state.currentGame = null;
    },
    setError: (state, action: PayloadAction<string>) => {
      state.error = action.payload;
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchGames.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchGames.fulfilled, (state, action) => {
        state.loading = false;
        state.games = action.payload;
      })
      .addCase(fetchGames.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch games';
      })
      .addCase(createGame.fulfilled, (state, action) => {
        state.games.unshift(action.payload);
      })
      .addCase(startGameSession.fulfilled, (state, action) => {
        state.currentSession = action.payload;
      });
  },
});

export const { setCurrentGame, clearCurrentGame, setError, clearError } = gameSlice.actions;
export default gameSlice.reducer;
```

#### 5.4.3 API Service
```typescript
// frontend/src/services/gameAPI.ts
import axios from 'axios';
import { Game, Question, GameSession, LeaderboardEntry } from '../types/game';

const API_BASE_URL = process.env.REACT_APP_BACKEND_URL || 'http://localhost:8001';

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor to add auth token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('authToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export const gameAPI = {
  // Game management
  getGames: (params: { skip?: number; limit?: number; filter_by?: string }) =>
    api.get('/api/games', { params }),
  
  getGame: (gameId: string) =>
    api.get(`/api/games/${gameId}`),
  
  createGame: (gameData: Partial<Game>) =>
    api.post('/api/games', gameData),
  
  updateGame: (gameId: string, gameData: Partial<Game>) =>
    api.put(`/api/games/${gameId}`, gameData),
  
  deleteGame: (gameId: string) =>
    api.delete(`/api/games/${gameId}`),
  
  // Question management
  getGameQuestions: (gameId: string) =>
    api.get(`/api/games/${gameId}/questions`),
  
  addQuestion: (gameId: string, questionData: Partial<Question>) =>
    api.post(`/api/games/${gameId}/questions`, questionData),
  
  updateQuestion: (questionId: string, questionData: Partial<Question>) =>
    api.put(`/api/questions/${questionId}`, questionData),
  
  deleteQuestion: (questionId: string) =>
    api.delete(`/api/questions/${questionId}`),
  
  // Game sessions
  startGameSession: (gameId: string) =>
    api.post(`/api/games/${gameId}/start`),
  
  submitAnswer: (sessionId: string, questionId: string, answer: string, timeTaken?: number) =>
    api.post(`/api/sessions/${sessionId}/answer`, {
      question_id: questionId,
      answer,
      time_taken: timeTaken,
    }),
  
  completeGameSession: (sessionId: string, totalTime: number) =>
    api.post(`/api/sessions/${sessionId}/complete`, { total_time: totalTime }),
  
  // Leaderboards
  getGameLeaderboard: (gameId: string, type: 'class' | 'school' | 'subject' = 'school') =>
    api.get(`/api/games/${gameId}/leaderboard`, { params: { type } }),
  
  getGlobalLeaderboard: () =>
    api.get('/api/leaderboard/global'),
};
```

#### 5.4.4 React Components

##### Admin Dashboard Component
```tsx
// frontend/src/components/admin/GamesDashboard.tsx
import React, { useEffect, useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { PlusIcon, ChartBarIcon, UserGroupIcon, GameController2Icon } from '@heroicons/react/24/outline';
import { fetchGames, createGame } from '../../store/slices/gameSlice';
import { RootState, AppDispatch } from '../../store';
import GameCard from './GameCard';
import CreateGameModal from './CreateGameModal';
import StatsCard from '../common/StatsCard';

const GamesDashboard: React.FC = () => {
  const dispatch = useDispatch<AppDispatch>();
  const { games, loading, error } = useSelector((state: RootState) => state.games);
  const [showCreateModal, setShowCreateModal] = useState(false);
  const [filter, setFilter] = useState('all');

  useEffect(() => {
    dispatch(fetchGames({ limit: 20 }));
  }, [dispatch]);

  const handleCreateGame = async (gameData: any) => {
    try {
      await dispatch(createGame(gameData)).unwrap();
      setShowCreateModal(false);
    } catch (error) {
      console.error('Failed to create game:', error);
    }
  };

  const filteredGames = games.filter(game => {
    if (filter === 'all') return true;
    if (filter === 'published') return game.is_published;
    if (filter === 'draft') return !game.is_published;
    return game.category === filter;
  });

  const stats = {
    totalGames: games.length,
    publishedGames: games.filter(g => g.is_published).length,
    totalPlayers: games.reduce((acc, game) => acc + (game.player_count || 0), 0),
    totalQuestions: games.reduce((acc, game) => acc + (game.question_count || 0), 0),
  };

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
      </div>
    );
  }

  return (
    <div className="space-y-6">
      {/* Header */}
      <div className="flex justify-between items-center">
        <div>
          <h1 className="text-3xl font-bold text-gray-900">Educational Games</h1>
          <p className="text-gray-600">Manage and monitor your educational games</p>
        </div>
        <button
          onClick={() => setShowCreateModal(true)}
          className="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg flex items-center space-x-2"
        >
          <PlusIcon className="h-5 w-5" />
          <span>Create Game</span>
        </button>
      </div>

      {/* Stats Cards */}
      <div className="grid grid-cols-1 md:grid-cols-4 gap-6">
        <StatsCard
          title="Total Games"
          value={stats.totalGames}
          icon={GameController2Icon}
          color="blue"
        />
        <StatsCard
          title="Published Games"
          value={stats.publishedGames}
          icon={ChartBarIcon}
          color="green"
        />
        <StatsCard
          title="Total Players"
          value={stats.totalPlayers}
          icon={UserGroupIcon}
          color="purple"
        />
        <StatsCard
          title="Questions Created"
          value={stats.totalQuestions}
          icon={ChartBarIcon}
          color="orange"
        />
      </div>

      {/* Filters */}
      <div className="flex space-x-4">
        {['all', 'published', 'draft', 'quiz', 'math', 'language', 'memory'].map((filterType) => (
          <button
            key={filterType}
            onClick={() => setFilter(filterType)}
            className={`px-4 py-2 rounded-lg capitalize ${
              filter === filterType
                ? 'bg-blue-600 text-white'
                : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
            }`}
          >
            {filterType}
          </button>
        ))}
      </div>

      {/* Games Grid */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {filteredGames.map((game) => (
          <GameCard key={game.id} game={game} />
        ))}
      </div>

      {/* Empty State */}
      {filteredGames.length === 0 && (
        <div className="text-center py-12">
          <GameController2Icon className="mx-auto h-12 w-12 text-gray-400" />
          <h3 className="mt-2 text-sm font-medium text-gray-900">No games found</h3>
          <p className="mt-1 text-sm text-gray-500">
            Get started by creating your first educational game.
          </p>
          <div className="mt-6">
            <button
              onClick={() => setShowCreateModal(true)}
              className="inline-flex items-center px-4 py-2 border border-transparent shadow-sm text-sm font-medium rounded-md text-white bg-blue-600 hover:bg-blue-700"
            >
              <PlusIcon className="-ml-1 mr-2 h-5 w-5" />
              Create Game
            </button>
          </div>
        </div>
      )}

      {/* Create Game Modal */}
      {showCreateModal && (
        <CreateGameModal
          onClose={() => setShowCreateModal(false)}
          onSubmit={handleCreateGame}
        />
      )}
    </div>
  );
};

export default GamesDashboard;
```

##### Student Games Component
```tsx
// frontend/src/components/student/StudentGames.tsx
import React, { useEffect, useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { PlayIcon, TrophyIcon, ClockIcon } from '@heroicons/react/24/outline';
import { fetchGames } from '../../store/slices/gameSlice';
import { RootState, AppDispatch } from '../../store';
import { Game } from '../../types/game';

const StudentGames: React.FC = () => {
  const dispatch = useDispatch<AppDispatch>();
  const { games, loading } = useSelector((state: RootState) => state.games);
  const [selectedCategory, setSelectedCategory] = useState('all');

  useEffect(() => {
    // Fetch only published games for students
    dispatch(fetchGames({ filter_by: 'published' }));
  }, [dispatch]);

  const filteredGames = games.filter(game => {
    if (selectedCategory === 'all') return true;
    return game.category === selectedCategory;
  });

  const categories = [
    { id: 'all', name: 'All Games', icon: '🎮' },
    { id: 'quiz', name: 'Quiz Games', icon: '📚' },
    { id: 'math', name: 'Math Games', icon: '🧮' },
    { id: 'language', name: 'Language Games', icon: '🌍' },
    { id: 'memory', name: 'Memory Games', icon: '🧠' },
  ];

  const getDifficultyColor = (difficulty: string) => {
    switch (difficulty) {
      case 'easy': return 'bg-green-100 text-green-800';
      case 'medium': return 'bg-yellow-100 text-yellow-800';
      case 'hard': return 'bg-red-100 text-red-800';
      default: return 'bg-gray-100 text-gray-800';
    }
  };

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
      </div>
    );
  }

  return (
    <div className="space-y-6">
      {/* Header */}
      <div className="text-center">
        <h1 className="text-3xl font-bold text-gray-900">Educational Games</h1>
        <p className="text-gray-600 mt-2">Choose a game to play and test your knowledge!</p>
      </div>

      {/* Category Filters */}
      <div className="flex flex-wrap justify-center gap-4">
        {categories.map((category) => (
          <button
            key={category.id}
            onClick={() => setSelectedCategory(category.id)}
            className={`px-6 py-3 rounded-lg flex items-center space-x-2 transition-colors ${
              selectedCategory === category.id
                ? 'bg-blue-600 text-white'
                : 'bg-white text-gray-700 hover:bg-gray-50 border border-gray-300'
            }`}
          >
            <span className="text-xl">{category.icon}</span>
            <span className="font-medium">{category.name}</span>
          </button>
        ))}
      </div>

      {/* Games Grid */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {filteredGames.map((game) => (
          <div
            key={game.id}
            className="bg-white rounded-xl shadow-lg overflow-hidden hover:shadow-xl transition-shadow"
          >
            {/* Game Header */}
            <div className="bg-gradient-to-r from-blue-500 to-purple-600 px-6 py-4">
              <div className="flex justify-between items-start">
                <h3 className="text-xl font-bold text-white">{game.title}</h3>
                <span className={`px-2 py-1 rounded-full text-xs font-medium ${getDifficultyColor(game.difficulty)}`}>
                  {game.difficulty}
                </span>
              </div>
              <p className="text-blue-100 mt-2 text-sm">{game.description}</p>
            </div>

            {/* Game Content */}
            <div className="p-6">
              <div className="flex items-center justify-between text-sm text-gray-600 mb-4">
                <div className="flex items-center space-x-1">
                  <TrophyIcon className="h-4 w-4" />
                  <span>{game.points_per_question} pts/question</span>
                </div>
                {game.time_limit && (
                  <div className="flex items-center space-x-1">
                    <ClockIcon className="h-4 w-4" />
                    <span>{game.time_limit} min</span>
                  </div>
                )}
              </div>

              <div className="flex items-center justify-between text-sm text-gray-500 mb-4">
                <span>{game.question_count} questions</span>
                <span>{game.player_count} players</span>
              </div>

              <button
                onClick={() => {/* Navigate to game play */}}
                className="w-full bg-blue-600 hover:bg-blue-700 text-white py-3 px-4 rounded-lg flex items-center justify-center space-x-2 transition-colors"
              >
                <PlayIcon className="h-5 w-5" />
                <span>Play Game</span>
              </button>
            </div>
          </div>
        ))}
      </div>

      {/* Empty State */}
      {filteredGames.length === 0 && (
        <div className="text-center py-12">
          <div className="text-6xl mb-4">🎮</div>
          <h3 className="text-lg font-medium text-gray-900">No games available</h3>
          <p className="text-gray-500 mt-2">
            Check back later for new games in this category!
          </p>
        </div>
      )}
    </div>
  );
};

export default StudentGames;
```

##### Leaderboard Component
```tsx
// frontend/src/components/common/Leaderboard.tsx
import React, { useEffect, useState } from 'react';
import { TrophyIcon, MedalIcon, AwardIcon } from '@heroicons/react/24/outline';
import { gameAPI } from '../../services/gameAPI';
import { LeaderboardEntry } from '../../types/game';

interface LeaderboardProps {
  gameId?: string;
  type?: 'class' | 'school' | 'subject';
  limit?: number;
}

const Leaderboard: React.FC<LeaderboardProps> = ({ 
  gameId, 
  type = 'school', 
  limit = 10 
}) => {
  const [leaderboard, setLeaderboard] = useState<LeaderboardEntry[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchLeaderboard = async () => {
      try {
        setLoading(true);
        const response = gameId 
          ? await gameAPI.getGameLeaderboard(gameId, type)
          : await gameAPI.getGlobalLeaderboard();
        
        setLeaderboard(response.data.slice(0, limit));
      } catch (error) {
        console.error('Failed to fetch leaderboard:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchLeaderboard();
  }, [gameId, type, limit]);

  const getRankIcon = (rank: number) => {
    switch (rank) {
      case 1: return <TrophyIcon className="h-6 w-6 text-yellow-500" />;
      case 2: return <MedalIcon className="h-6 w-6 text-gray-400" />;
      case 3: return <AwardIcon className="h-6 w-6 text-orange-500" />;
      default: return <span className="text-lg font-bold text-gray-600">#{rank}</span>;
    }
  };

  const getRankBackground = (rank: number) => {
    switch (rank) {
      case 1: return 'bg-gradient-to-r from-yellow-400 to-yellow-600';
      case 2: return 'bg-gradient-to-r from-gray-300 to-gray-500';
      case 3: return 'bg-gradient-to-r from-orange-400 to-orange-600';
      default: return 'bg-white';
    }
  };

  if (loading) {
    return (
      <div className="animate-pulse space-y-4">
        {[...Array(5)].map((_, i) => (
          <div key={i} className="flex items-center space-x-4 p-4 bg-gray-100 rounded-lg">
            <div className="w-12 h-12 bg-gray-300 rounded-full"></div>
            <div className="flex-1 space-y-2">
              <div className="h-4 bg-gray-300 rounded w-1/4"></div>
              <div className="h-3 bg-gray-300 rounded w-1/2"></div>
            </div>
            <div className="h-6 bg-gray-300 rounded w-16"></div>
          </div>
        ))}
      </div>
    );
  }

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <h2 className="text-xl font-bold text-gray-900">
          🏆 {type.charAt(0).toUpperCase() + type.slice(1)} Leaderboard
        </h2>
        <div className="text-sm text-gray-500">
          Top {leaderboard.length} performers
        </div>
      </div>

      <div className="space-y-2">
        {leaderboard.map((entry, index) => {
          const rank = index + 1;
          const isTopThree = rank <= 3;
          
          return (
            <div
              key={entry.id}
              className={`flex items-center space-x-4 p-4 rounded-lg transition-all hover:shadow-md ${
                isTopThree 
                  ? `${getRankBackground(rank)} text-white` 
                  : 'bg-white border border-gray-200 hover:border-gray-300'
              }`}
            >
              {/* Rank */}
              <div className="flex-shrink-0 w-12 flex justify-center">
                {getRankIcon(rank)}
              </div>

              {/* Student Photo */}
              <div className="flex-shrink-0">
                <img
                  src={entry.student_photo || '/default-avatar.png'}
                  alt={entry.student_name}
                  className="w-12 h-12 rounded-full object-cover border-2 border-white shadow-sm"
                />
              </div>

              {/* Student Info */}
              <div className="flex-1 min-w-0">
                <div className={`font-semibold truncate ${isTopThree ? 'text-white' : 'text-gray-900'}`}>
                  {entry.student_name}
                </div>
                <div className={`text-sm truncate ${isTopThree ? 'text-gray-100' : 'text-gray-600'}`}>
                  {entry.class_name} • {entry.total_attempts} attempts
                </div>
              </div>

              {/* Score */}
              <div className="flex-shrink-0 text-right">
                <div className={`text-lg font-bold ${isTopThree ? 'text-white' : 'text-blue-600'}`}>
                  {entry.best_score}
                </div>
                {entry.best_time && (
                  <div className={`text-xs ${isTopThree ? 'text-gray-100' : 'text-gray-500'}`}>
                    {Math.floor(entry.best_time / 60)}:{(entry.best_time % 60).toString().padStart(2, '0')}
                  </div>
                )}
              </div>
            </div>
          );
        })}
      </div>

      {leaderboard.length === 0 && (
        <div className="text-center py-8">
          <TrophyIcon className="mx-auto h-12 w-12 text-gray-400" />
          <h3 className="mt-2 text-sm font-medium text-gray-900">No leaderboard data</h3>
          <p className="mt-1 text-sm text-gray-500">
            Play some games to see rankings here!
          </p>
        </div>
      )}
    </div>
  );
};

export default Leaderboard;
```

---

## 6. Installation & Setup Guide

### 6.1 Current PHP System Setup

#### Step 1: Database Installation
```bash
# Execute the complete database script
mysql -u root -p your_database_name < /app/complete_educational_games.sql

# Verify tables were created
mysql -u root -p your_database_name -e "SHOW TABLES LIKE '%game%';"
```

#### Step 2: Permission Configuration
```sql
-- Fix permission issues (run in MySQL)
-- Ensure Educational Games permissions exist
INSERT IGNORE INTO permission_group (name, short_code, is_active) 
VALUES ('Game Builder', 'game_builder', 1);

-- Get the group ID
SET @group_id = (SELECT id FROM permission_group WHERE short_code = 'game_builder');

-- Add permission categories
INSERT IGNORE INTO permission_category (perm_group_id, name, short_code, enable_view, enable_add, enable_edit, enable_delete) VALUES
(@group_id, 'Games Management', 'games_management', 1, 1, 1, 1),
(@group_id, 'Game Results', 'game_results', 1, 0, 0, 0),
(@group_id, 'Game Analytics', 'game_analytics', 1, 0, 0, 0);

-- Grant permissions to Super Admin (adjust role_id as needed)
INSERT IGNORE INTO roles_permissions (role_id, perm_cat_id, can_view, can_add, can_edit, can_delete)
SELECT 7, pc.id, pc.enable_view, pc.enable_add, pc.enable_edit, pc.enable_delete
FROM permission_category pc 
WHERE pc.short_code IN ('games_management', 'game_results', 'game_analytics');

-- Enable student module
INSERT IGNORE INTO permission_student (name, short_code, student, parent, is_active) 
VALUES ('Educational Games', 'student_games', 1, 0, 1);
```

#### Step 3: Menu Integration
Add to admin sidebar (`/application/views/layout/sidebar.php`):
```php
<?php if ($this->rbac->hasPrivilege('games_management', 'can_view')) { ?>
<li class="treeview <?php echo set_Topmenu('gamebuilder'); ?>">
    <a href="#">
        <i class="fa fa-gamepad"></i>
        <span>Educational Games</span>
        <span class="pull-right-container">
            <i class="fa fa-angle-left pull-right"></i>
        </span>
    </a>
    <ul class="treeview-menu">
        <li><a href="<?php echo base_url(); ?>gamebuilder"><i class="fa fa-circle-o"></i> Manage Games</a></li>
        <li><a href="<?php echo base_url(); ?>gamebuilder/create"><i class="fa fa-circle-o"></i> Create Game</a></li>
        <li><a href="<?php echo base_url(); ?>gamebuilder/results"><i class="fa fa-circle-o"></i> View Results</a></li>
        <li><a href="<?php echo base_url(); ?>gamebuilder/leaderboard"><i class="fa fa-circle-o"></i> Leaderboard</a></li>
    </ul>
</li>
<?php } ?>
```

Add to student sidebar (`/application/views/layout/student/header.php`):
```php
<?php if ($this->module_lib->hasActive('student_games')) { ?>
<li class="<?php echo set_Topmenu('Gamebuilder'); ?>">
    <a href="<?php echo base_url(); ?>user/gamebuilder/student_games">
        <i class="fa fa-gamepad ftlayer"></i> 
        <span>Educational Games</span>
    </a>
</li>
<?php } ?>
```

### 6.2 Modern React/FastAPI Setup

#### Step 1: Backend Setup
```bash
# Create project structure
mkdir educational-games-modern
cd educational-games-modern
mkdir backend frontend

# Backend setup
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install fastapi uvicorn motor pymongo python-jose passlib python-multipart

# Create requirements.txt
cat > requirements.txt << EOF
fastapi==0.104.1
uvicorn==0.24.0
motor==3.3.2
pymongo==4.6.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
pydantic==2.5.0
python-dotenv==1.0.0
EOF

# Install dependencies
pip install -r requirements.txt
```

#### Step 2: Frontend Setup
```bash
# Frontend setup
cd ../frontend
npx create-react-app . --template typescript
npm install @reduxjs/toolkit react-redux @heroicons/react tailwindcss axios

# Setup Tailwind CSS
npx tailwindcss init -p
```

#### Step 3: Environment Configuration
Backend `.env`:
```env
MONGO_URL=mongodb://localhost:27017
DATABASE_NAME=educational_games
SECRET_KEY=your-secret-key-here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Frontend `.env`:
```env
REACT_APP_BACKEND_URL=http://localhost:8001
```

#### Step 4: Start Services
```bash
# Start backend
cd backend
uvicorn server:app --reload --port 8001

# Start frontend (in new terminal)
cd frontend
npm start
```

---

## 7. Troubleshooting Common Issues

### 7.1 Permission Issues

#### Buttons Not Showing
```sql
-- Debug: Check if permissions exist
SELECT pg.name as group_name, pc.name as permission_name, pc.short_code
FROM permission_category pc
JOIN permission_group pg ON pg.id = pc.perm_group_id
WHERE pc.short_code LIKE '%game%';

-- Fix: Create missing permissions
INSERT IGNORE INTO permission_group (name, short_code, is_active) 
VALUES ('Game Builder', 'game_builder', 1);
```

#### Access Denied Errors
```php
// Debug user permissions in controller
public function index() {
    $user_data = $this->customlib->getUserData();
    echo "<pre>User Data: "; print_r($user_data); echo "</pre>";
    
    $has_permission = $this->rbac->hasPrivilege('games_management', 'can_view');
    echo "<pre>Has Permission: "; var_dump($has_permission); echo "</pre>";
    die();
}
```

### 7.2 Routing Issues

#### Wrong Controller Called
Check routes configuration:
```php
// In application/config/routes.php
$route['gamebuilder'] = 'admin/gamebuilder/index';
$route['gamebuilder/(.+)'] = 'admin/gamebuilder/$1';
$route['user/games'] = 'user/gamebuilder/student_games';
```

#### Student Login Redirect
```php
// Fix authentication check in base controller
protected function check_auth() {
    if (!$this->session->userdata('logged_in')) {
        redirect('login');
    }
    
    // Debug role
    $role = $this->session->userdata('role');
    if ($role === 'student' && strpos(current_url(), '/admin/') !== false) {
        redirect('user/dashboard');  // Redirect students away from admin
    }
}
```

### 7.3 Database Issues

#### Missing Tables
```sql
-- Check if all game tables exist
SELECT table_name 
FROM information_schema.tables 
WHERE table_schema = 'your_database_name' 
AND table_name LIKE '%game%';

-- Recreate if missing
SOURCE /app/complete_educational_games.sql;
```

#### Foreign Key Errors
```sql
-- Check existing foreign key constraints
SELECT 
    TABLE_NAME,
    COLUMN_NAME,
    CONSTRAINT_NAME,
    REFERENCED_TABLE_NAME,
    REFERENCED_COLUMN_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE REFERENCED_TABLE_SCHEMA = 'your_database_name'
AND TABLE_NAME LIKE '%game%';
```

### 7.4 Modern System Issues

#### CORS Errors
```python
# Add CORS middleware in FastAPI
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

#### MongoDB Connection Issues
```python
# Test MongoDB connection
from motor.motor_asyncio import AsyncIOMotorClient

async def test_connection():
    client = AsyncIOMotorClient("mongodb://localhost:27017")
    try:
        await client.admin.command('ping')
        print("MongoDB connection successful")
    except Exception as e:
        print(f"MongoDB connection failed: {e}")
```

---

## 8. Summary and Recommendations

### Current System Strengths
✅ **Comprehensive Database Design**: Well-structured with proper relationships  
✅ **RBAC Integration**: Proper permission system  
✅ **Multiple Game Types**: Quiz, Math, Language, Memory games  
✅ **Analytics & Leaderboards**: Complete tracking system  
✅ **Session Management**: Multi-attempt support  

### Issues to Address
❌ **Permission Configuration**: Ensure proper RBAC setup  
❌ **Menu Integration**: Fix sidebar menu visibility  
❌ **Route Configuration**: Prevent student/admin crossover  
❌ **Database Integrity**: Ensure all tables and triggers exist  

### Modern System Benefits
🚀 **Better Performance**: FastAPI + React for modern web standards  
🚀 **Real-time Features**: WebSocket support for live leaderboards  
🚀 **Mobile Responsive**: Better mobile experience with React  
🚀 **Scalability**: MongoDB for better scaling  
🚀 **Developer Experience**: TypeScript for better code maintainability  

### Implementation Priority
1. **Fix Current Issues**: Resolve permission and routing problems
2. **Complete Documentation**: Ensure all features are documented
3. **Modern Migration**: Gradually migrate to React/FastAPI system
4. **Testing**: Implement comprehensive testing for both systems

This documentation provides both the current system analysis and a modern implementation path, giving you flexibility to fix immediate issues while planning for future improvements.
