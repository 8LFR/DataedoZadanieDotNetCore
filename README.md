# DataedoZadanieDotNetCore

## Treść zadania:
Kolega rozszerzył kontroler w aplikacji (.NET Core) o akcję usuwania pojedynczego
użytkownika. Poprosił Cię o code-review.
Przy założeniu, że kod jest składniowo poprawny i kompiluje się, na co zwrócisz uwagę
przy recenzji tego fragmentu?

```
    [HttpPost("delete/{id}")]
    public void Delete(uint id)
    {
        User user = _context.Users.FirstOrDefault(user => user.Id == id);
        _context.Users.Remove(user);
        _context.SaveChanges();
        Debug.WriteLine($"The user with Login={user.login} has been deleted.");
        return Ok();
    }
```

## Rozwiązanie:
1. Brak obsługi błędów - w przypadku kiedy wartość zmiennej wyniesie null, następna linijka rzuci błędem NullReferenceException.
2. Niepoprawne zapytanie HTTP - zamiast POST powinniśmy użyć DELETE.
3. Niepoprawny zwracany typ - metoda powinna zwracać odpowiednią odpowiedź HTTP, dla kontrolera odpowiednim zwracanym typem będzie IActionResult.
4. Użycie Loggera - mały błąd, ale warty zwrócenia uwagi. Logger pozwala na znacznie lepsze wypisywanie logów, posiada poziomy logów (error, info itp.) lub może przestawić je w postaci np. JSONa.
5. Autoryzacja - bezpośrednie działanie na bazie danych, a zwłaszcza usuwanie rekordów z niej powinno być autoryzowane aby z funkcji tej mógł użyć tylko odpowiedni użytkownik np. admin.
6. Asynchroniczność - dodanie asynchroniczności pozwoli aplikacji m.in. na jednoczesną obsługę większej liczby żądań oraz zapewni lepszą responsywność.

Po zastosowaniu tych komentarzy kod powinien wyglądać niewięcej tak:
```
    [HttpDelete("delete/{id}")]
    public async Task<IActionResult> Delete(uint id)
    {
        var user = await _context.Users.FirstOrDefaultAsync(user => user.Id == id);
        if (user == null)
        {
            _logger.LogWarning($"User with ID={id} not found.");
            return NotFound($"User with ID={id} not found.");
        }

        _context.Users.Remove(user);
        await _context.SaveChangesAsync();

        _logger.LogInformation($"User with Login: {user.login} has been deleted.");
        return Ok();
    }
```
