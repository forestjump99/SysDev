# Domain model completeness

"Domain model completeness" är när alla affärsregler (domänlogik) finns i domänklasserna. Ej i Controllers.

## Exempel - Ändra användarens e-postadress.
Så här ser domänmodellsklassen User ut:

```csharp
public class User : Entity
{
    public Company Company { get; private set; }
    public string Email { get; private set; }

    public Result ChangeEmail(string newEmail)
    {
        if (Company.IsEmailCorporate(newEmail) == false)
            return Result.Failure("Incorrect email domain");

        Email = newEmail;

        return Result.Success();
    }
}

public class Company : Entity
{
    public string DomainName { get; }

    public bool IsEmailCorporate(string email)
    {
        string emailDomain = email.Split('@')[1];
        return emailDomain == DomainName;
    }
}
```
Och det här är Controllern som orkestrerar anropen:

```csharp
public class UserController
{
    public string ChangeEmail(int userId, string newEmail)
    {
        User user = _userRepository.GetById(userId);

        Result result = user.ChangeEmail(newEmail);
        if (result.IsFailure)
            return result.Error;

        _userRepository.Save(user);

        return "OK";
    }
}
```
