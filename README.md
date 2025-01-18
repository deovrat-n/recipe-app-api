@extend_schema_view(
    list=extend_schema(
        parameters=[
            OpenApiParameter(
                'tags',
                OpenApiTypes.STR,
                description='Comma separated list of tag IDs to filter',
            ),
            OpenApiParameter(
                'ingredients',
                OpenApiTypes.STR,
                description='Comma separated list of ingredient IDs to filter',
            ),
        ]
    )
)


What It Does
The @extend_schema_view decorator is applied to a class-based view (e.g., RecipeViewSet) to customize or extend the API schema for specific actions (e.g., list). In this case:

list Action Customization:

Adds query parameters (tags and ingredients) to the schema for the list endpoint.
Query Parameters:

tags: A comma-separated string representing tag IDs to filter recipes.
ingredients: A comma-separated string representing ingredient IDs to filter recipes.
Parameter Configuration:

OpenApiParameter is used to define each query parameter, specifying:
Name: 'tags' or 'ingredients'.
Type: OpenApiTypes.STR (string).
Description: A helpful explanation of what the parameter does.

----------------------------------------------
def get_queryset(self):
        """Filter queryset to authenticated user."""
        assigned_only = bool(
            int(self.request.query_params.get('assigned_only', 0))
        )
        queryset = self.queryset
        if assigned_only:
            queryset = queryset.filter(recipe__isnull=False)

        return queryset.filter(
            user=self.request.user
        ).order_by('-name').distinct()
----------------------------------------------------------------
The line queryset.filter(recipe__isnull=False) is a Django ORM filter expression that checks whether a particular field (recipe) is not null in the database. Here's a detailed breakdown of what it does:

Understanding the Query
recipe__isnull:

The recipe field refers to a foreign key or reverse relationship between the model being queried and the Recipe model.
__isnull is a query lookup in Django that checks whether the field is NULL in the database.
False:

isnull=False means "the recipe field should NOT be null."
In plain terms, this filters the queryset to include only records where there is an associated Recipe.

When and Why It's Used
This query is likely being applied in a situation where:

The current model (e.g., Tag, Ingredient) has a relationship with the Recipe model.
The goal is to filter for instances (e.g., tags or ingredients) that are assigned to at least one recipe.
For example:

If querying Tag objects:

This will return tags that are assigned to one or more recipes.
Tags that are not linked to any recipe are excluded from the queryset.
If querying Ingredient objects:

This will return ingredients that are used in at least one recipe.
