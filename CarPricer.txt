using System;
using NUnit.Framework;

namespace CarPricer
{
  public class Car
  {
    public decimal PurchaseValue { get; set; }
    public int AgeInMonths { get; set; }
    public int NumberOfMiles { get; set; }
    public int NumberOfPreviousOwners { get; set; }
    public int NumberOfCollisions { get; set; }
  }

  public class PriceDeterminator
  {
    public decimal DetermineCarPrice(Car car)
    {
      decimal finalValue = ageDepreciation(car, car.PurchaseValue);
      finalValue = mileageDepreciation(car, finalValue);
      finalValue = ownerDepreciation(car, finalValue);
      finalValue = collisionDepreciation(car, finalValue);
      finalValue = ownerBonus(car, finalValue);

      return finalValue;
    }

    public decimal ageDepreciation(Car car, decimal value)
    {
      // For every month under 120 (10 years), reduce value by .005%
      const int maxAgeLimit = 10 * 12; // 10 years in months
      const decimal percentReducedByAge = .005M; // Half of 1%

      int ageInMonths = car.AgeInMonths > maxAgeLimit ? maxAgeLimit : car.AgeInMonths;
      return value - (ageInMonths * percentReducedByAge * value);
    }

    public decimal mileageDepreciation(Car car, decimal value)
    {
      // For every 1000 miles (max of 150k miles), reduce value by .005%
      const int maxMiles = 150000;
      const int mileageRate = 1000;
      const decimal percentReducedByMiles = .002M; // 1/5 of 1%

      int milesToUse = car.NumberOfMiles > maxMiles ? maxMiles : car.NumberOfMiles;
      return value - ((milesToUse / mileageRate) * percentReducedByMiles * value);
    }

    public decimal ownerDepreciation(Car car, decimal value)
    {
      // If car had more than 2 previous owners, reduce value by 25%
      const int minPreviousOwners = 2;
      const decimal percentReducedByOwners = .25M; // 25%

      if (car.NumberOfPreviousOwners > minPreviousOwners)
      {
        return value - (value * percentReducedByOwners);
      }
      return value;
    }

    public decimal collisionDepreciation(Car car, decimal value)
    {
      // Each collision reduces vehicle value by 2%
      const int maxCollisions = 5;
      const decimal percentReducedByCollision = .02M; // 2%

      int collisions = car.NumberOfCollisions > maxCollisions ? maxCollisions : car.NumberOfCollisions;
      return value - (collisions * percentReducedByCollision * value);
    }

    public decimal ownerBonus(Car car, decimal value)
    {
      // Owner receives 10% bonus if car had no previous owners
      const decimal percentPreviousOwnerBonus = .1M; // 10%

      if (car.NumberOfPreviousOwners == 0)
      {
        return value + (value * percentPreviousOwnerBonus);
      }
      return value;
    }
  }


  public class UnitTests
  {

    public void CalculateCarValue()
    {
      AssertCarValue(25313.40m, 35000m, 3 * 12, 50000, 1, 1);
      AssertCarValue(19688.20m, 35000m, 3 * 12, 150000, 1, 1);
      AssertCarValue(19688.20m, 35000m, 3 * 12, 250000, 1, 1);
      AssertCarValue(20090.00m, 35000m, 3 * 12, 250000, 1, 0);
      AssertCarValue(21657.02m, 35000m, 3 * 12, 250000, 0, 1);
    }

    private static void AssertCarValue(decimal expectValue, decimal purchaseValue,
    int ageInMonths, int numberOfMiles, int numberOfPreviousOwners, int
    numberOfCollisions)
    {
      Car car = new Car
      {
        AgeInMonths = ageInMonths,
        NumberOfCollisions = numberOfCollisions,
        NumberOfMiles = numberOfMiles,
        NumberOfPreviousOwners = numberOfPreviousOwners,
        PurchaseValue = purchaseValue
      };
      PriceDeterminator priceDeterminator = new PriceDeterminator();
      var carPrice = priceDeterminator.DetermineCarPrice(car);
      Assert.AreEqual(expectValue, carPrice);
    }
  }
}
