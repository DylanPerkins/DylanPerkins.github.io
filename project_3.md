---
layout: default
title: "Project 3: Anime Tracker"
---

## Project 3: Anime Tracker

**Competency:** Databases

## Overview

This project was originally created in February 2023 as the final project and capstone for a backend bootcamp course at Promineo Tech. It is a Java-based back-end application that uses Spring Boot and MySQL to provide a RESTful API for users to review, rate, and track anime, as well as create and manage anime lists.

The project was refactored in 2026, with my main focus for enhancement being input validation, and fixing any severe security issues. I also wanted to expand my GET quesries to be a bit more user friendly, so I added two more GET endpoints.

Repository Link: [github.com/DylanPerkins/Anime_Reviews](https://github.com/DylanPerkins/Anime_Reviews)

## Technologies Used

- **Java 17**
- **Spring Boot**
- **MySQL**
- **Spring JPA** (data access)
- **Jakarta**
- **Lombok**
- **Maven**
- **RESTful API**

## Database Schema

The application has a relational MySQL database with the following tables and relationships:

- **anime** - Anime details like title, studio and episode count
- **tags** - Labels for anime
- **anime_tags** - Many-to-many join table for anime to tags
- **users** - User data like username
- **anime_users** - Many-to-many join table for users to anime
- **users_watched_anime / users_watching_anime / users_want_to_watch / users_wont_watch** - User anime tracking lists
- **anime_review** - Anime reviews with a rating and text, linked to both a user and an anime

## API Endpoints

The application exposes a full CRUD API including but not limited to:

| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| POST | `/anime-reviews/anime` | Create a new anime |
| PUT | `/anime-reviews/anime/{animeId}` | Update an anime |
| GET | `/anime-reviews/anime` | Get all anime, show tags |
| GET | `/anime-reviews/anime/{animeId}` | Get a specific anime, show tags |
| GET | `/anime-reviews/anime/search/name` | Search anime by name **(NEW)** |
| GET | `/anime-reviews/anime/search/tag` | Search anime by tag **(NEW)** |
| POST | `/anime-reviews/user` | Create a new user |
| GET | `/anime-reviews/user/{userId}` | Get a user's info |
| POST | `/anime-reviews/user/{userId}/anime/{animeId}/review` | Create a review |
| PUT | `/anime-reviews/user/{userId}/anime/{animeId}/review/{reviewId}` | Update a review |
| DELETE | `/anime-reviews/user/{userId}/anime/{animeId}/review/{reviewId}` | Delete a review |

Users can also manage anime lists (watched, watching, want-to-watch, won't-watch).

## Enhancements Made

### Enhancement 1: Input Validation

I added Jakarta Validation annotations across all Data classes to verify that invalid data is being rejected before it reaches the database or is processed. Before this enhancement, the API would accept any data without any verifications or . Examples of this would be blank titles, negative episode counts, or missing required fields.

For a coding example, in `AnimeData.java` I added validation constraints to the following fields:

```java
@NotBlank(message = "Title is required")
private String title;
@NotBlank(message = "Animation studio is required")
private String animationStudio;
@NotNull(message = "Episode count is required")
@Min(value = 1, message = "Episode count must be at least 1")
private Integer episodeCount;
```

And, for reviews in `AnimeReviewData.java`, I constrained ratings to a range that just makes sense, 0.0 to 10.0, and made the review text required:

```java
@DecimalMin(value = "0.0", message = "Rating must be at least 0.0")
@DecimalMax(value = "10.0", message = "Rating must be at most 10.0")
private double rating;
@NotBlank(message = "Review text is required")
private String reviewText;
```

I also added the `@Validated` annotation and `@Valid` on `@RequestBody` params so that Spring actually enforces these constraints, and does so automatically.

### Enhancement 2: Ownership Exploit Fix

During my coding review, I found an ownership vulnerability where the `deleteReviewById` method would delete any review by its ID without verifying that the review belonged to the user and anime that was passed in the URL path. This meant a user could delete another user's reviews inputting a valid review ID randomly.

I fixed this by adding an ownership check before allowing deletion:

```java
public void deleteReviewById(Long userId, Long animeId, Long reviewId) {
    // Find the user and anime by ID, see if they exist
    Users user = findUserById(userId);
    Anime anime = findAnimeById(animeId);

    // Find the review by ID, see if it exists
    AnimeReview review = animeReviewDAO.findById(reviewId)
            .orElseThrow(() -> new NoSuchElementException(
                "Review ID=" + reviewId + " not found"));

    // Check if the review belongs to the user and anime
    if (!review.getUser().getUserId().equals(user.getUserId())
            || !review.getAnime().getAnimeId().equals(anime.getAnimeId())) {
        throw new IllegalArgumentException(
            "Review ID=" + reviewId + " does not belong to User ID="
            + userId + " and Anime ID=" + animeId);
    }

    // If ownership is true, delete the review
    animeReviewDAO.delete(review);
}
```

### Enhancement 3: New Query Endpoints

The original API only allowed finding an anime by its ID or listing all anime in the database. I decided to make that approach easier, and added two new GET endpoints to achieve that:

**Search by anime name** — Partial names, not case sensitive:

```java
// Controller
@GetMapping("/anime/search/name")
public List<AnimeData> findAnimeByName(
        @NotBlank @RequestParam String name) {
    return animeService.findAnimeByName(name);
}
```

**Search by tag** — Exact only, not case sensitive:

```java
// Controller
@GetMapping("/anime/search/tag")
public List<AnimeData> findAnimeByTag(
        @NotBlank @RequestParam String tag) {
    return animeService.findAnimeByTag(tag);
}
```

These endpoints directly pass to the Service layer to the DAO layer, and the DAO automatically creates the SQL queries based on the method names:

```java
public interface AnimeDAO extends JpaRepository<Anime, Long> {
    List<Anime> findByTitleContainingIgnoreCase(String title);
    List<Anime> findByTags_TagNameIgnoreCase(String tagName);
}
```

## Skills Demonstrated

- Semi large relational database design with MySQL
- RESTful API design
- Secure coding practices via input validation and ownership checks

[Back to Home](./)
