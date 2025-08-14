<template>
  <div class="container py-5">
    <h1 class="mb-4 text-center">אודות הפרויקט</h1>
    <p class="lead text-center">
      אתר "המתכונים של סבתא ואחרים" פותח במסגרת פרויקט הקורס סביבות פיתוח באינטרנט. המערכת מאפשרת חיפוש, שיתוף וניהול מתכונים משפחתיים באווירה חמה ונגישה.
    </p>
    <p class="text-center">
      הפרויקט פותח על ידי צוות סטודנטים מסור, תוך הקפדה על עיצוב מודרני, חוויית משתמש איכותית ועבודה לפי סטנדרטים בתעשייה.
    </p>
    <div class="mt-4">
      <h3 class="text-center mb-3">תרגילים קודמים</h3>
      <ul class="list-group list-group-flush mx-auto" style="max-width: 500px;">
        <li class="list-group-item">
          <a href="https://wed-2023.github.io/313188799/" target="_blank">תרגיל 1- גיל שלישי (רוית)</a>
        </li>
        <li class="list-group-item">
          <a href="https://wed-2023.github.io/322685314/" target="_blank">תרגיל 1- גיל שלישי (נטלי)</a>
        </li>
        <li class="list-group-item">
          <a href=" https://wed-2023.github.io/assignment2-322685314_313188799_assignment2/" target="_blank">תרגיל 2 - בניית משחק</a>
        </li>
        <li class="list-group-item">
          <a href="/openapi-finalAPI.yaml" target="_blank">תרגיל 3.1 - API Documentation</a>
        </li>
        <li class="list-group-item">
          <a href="https://github.com/WED-2023/assignment3-2-322685314_313188799_assignment2" target="_blank">תרגיל 3.2 - Backend</a>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "AboutPage"
};
</script>

<style scoped>
h1 {
  font-weight: 700;
}
a {
  color: #007bff;
  text-decoration: none;
}
a:hover {
  text-decoration: underline;
}
</style>