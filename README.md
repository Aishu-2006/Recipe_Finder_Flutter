import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

void main() {
  runApp(const RecipeFinderApp());
}

class RecipeFinderApp extends StatelessWidget {
  const RecipeFinderApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Recipe Finder',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.orange,
        scaffoldBackgroundColor: Colors.orange.shade50,
        useMaterial3: true,
      ),
      home: const RecipeHomePage(),
    );
  }
}

class RecipeHomePage extends StatefulWidget {
  const RecipeHomePage({super.key});

  @override
  State<RecipeHomePage> createState() => _RecipeHomePageState();
}

class _RecipeHomePageState extends State<RecipeHomePage> {
  List<dynamic> recipes = [];
  bool isLoading = true;

  @override
  void initState() {
    super.initState();
    fetchRecipes('');
  }

  Future<void> fetchRecipes(String query) async {
    setState(() => isLoading = true);
    final url = Uri.parse('https://www.themealdb.com/api/json/v1/1/search.php?s=$query');
    try {
      final response = await http.get(url);
      if (response.statusCode == 200) {
        final data = json.decode(response.body);
        setState(() {
          recipes = data['meals'] ?? [];
          isLoading = false;
        });
      } else {
        setState(() {
          recipes = [];
          isLoading = false;
        });
      }
    } catch (e) {
      setState(() {
        recipes = [];
        isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('🍳 Recipe Finder'),
        centerTitle: true,
        backgroundColor: Colors.orange,
        foregroundColor: Colors.white,
        elevation: 3,
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(12),
            child: TextField(
              decoration: InputDecoration(
                hintText: 'Search for recipes...',
                prefixIcon: const Icon(Icons.search, color: Colors.orange),
                filled: true,
                fillColor: Colors.white,
                border: OutlineInputBorder(borderRadius: BorderRadius.circular(15)),
              ),
              onChanged: (value) => fetchRecipes(value),
            ),
          ),
          Expanded(
            child: isLoading
                ? const Center(child: CircularProgressIndicator(color: Colors.orange))
                : recipes.isEmpty
                    ? const Center(child: Text('No recipes found 😢'))
                    : Padding(
                        padding: const EdgeInsets.symmetric(horizontal: 10),
                        child: GridView.builder(
                          itemCount: recipes.length,
                          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                            crossAxisCount: 3, // ✅ 3 per row
                            crossAxisSpacing: 10,
                            mainAxisSpacing: 10,
                            // FIX: Changed childAspectRatio to provide more vertical space for content
                            childAspectRatio: 0.5, 
                          ),
                          itemBuilder: (context, index) {
                            final recipe = recipes[index];
                            return GestureDetector(
                              onTap: () {
                                Navigator.push(
                                  context,
                                  MaterialPageRoute(
                                    builder: (_) => RecipeDetailPage(recipe: recipe),
                                  ),
                                );
                              },
                              child: Card(
                                elevation: 4,
                                shape: RoundedRectangleBorder(
                                  borderRadius: BorderRadius.circular(16),
                                ),
                                clipBehavior: Clip.hardEdge,
                                shadowColor: Colors.orange.withAlpha((255 * 0.3).round()), 
                                child: Column(
                                  crossAxisAlignment: CrossAxisAlignment.center,
                                  children: [
                                    ClipRRect(
                                      borderRadius: const BorderRadius.vertical(top: Radius.circular(16)),
                                      child: AspectRatio(
                                        aspectRatio: 1, // ✅ Square image
                                        child: Image.network(
                                          recipe['strMealThumb'],
                                          fit: BoxFit.cover,
                                        ),
                                      ),
                                    ),
                                    Padding(
                                      padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 6),
                                      child: Text(
                                        recipe['strMeal'],
                                        maxLines: 2,
                                        overflow: TextOverflow.ellipsis,
                                        textAlign: TextAlign.center,
                                        style: const TextStyle(
                                          fontWeight: FontWeight.bold,
                                          fontSize: 14,
                                        ),
                                      ),
                                    ),
                                    Text(
                                      recipe['strArea'] ?? '',
                                      style: TextStyle(color: Colors.grey.shade600, fontSize: 12),
                                    ),
                                    const Spacer(),
                                    Padding(
                                      padding: const EdgeInsets.only(bottom: 8),
                                      child: ElevatedButton(
                                        style: ElevatedButton.styleFrom(
                                          backgroundColor: Colors.orange,
                                          minimumSize: const Size(60, 28),
                                          padding: EdgeInsets.zero,
                                          shape: RoundedRectangleBorder(
                                            borderRadius: BorderRadius.circular(10),
                                          ),
                                        ),
                                        onPressed: () {
                                          Navigator.push(
                                            context,
                                            MaterialPageRoute(
                                              builder: (_) => RecipeDetailPage(recipe: recipe),
                                            ),
                                          );
                                        },
                                        child: const Text(
                                          'View',
                                          style: TextStyle(color: Colors.white, fontSize: 12),
                                        ),
                                      ),
                                    ),
                                  ],
                                ),
                              ),
                            );
                          },
                        ),
                      ),
          ),
        ],
      ),
    );
  }
}

class RecipeDetailPage extends StatelessWidget {
  final Map<String, dynamic> recipe;
  const RecipeDetailPage({super.key, required this.recipe});

  @override
  Widget build(BuildContext context) {
    final ingredients = <String>[];
    for (int i = 1; i <= 20; i++) {
      final ingredient = recipe['strIngredient$i'];
      final measure = recipe['strMeasure$i'];
      if (ingredient != null && ingredient.isNotEmpty && measure != null && measure.isNotEmpty) {
        ingredients.add('$ingredient - $measure');
      }
    }

    return Scaffold(
      appBar: AppBar(
        title: Text(recipe['strMeal']),
        backgroundColor: Colors.orange,
        foregroundColor: Colors.white,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: LayoutBuilder(
          builder: (context, constraints) {
            return constraints.maxWidth > 600
                ? Row(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Expanded(
                        flex: 1,
                        child: ClipRRect(
                          borderRadius: BorderRadius.circular(15),
                          child: Image.network(recipe['strMealThumb'], fit: BoxFit.cover),
                        ),
                      ),
                      const SizedBox(width: 20),
                      Expanded(
                        flex: 2,
                        // Wrap the recipe details with SingleChildScrollView for larger screens
                        // to prevent overflow if the content exceeds the available height.
                        child: SingleChildScrollView(
                          child: recipeDetails(ingredients),
                        ),
                      ),
                    ],
                  )
                : SingleChildScrollView(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        ClipRRect(
                          borderRadius: BorderRadius.circular(15),
                          child: Image.network(recipe['strMealThumb']),
                        ),
                        const SizedBox(height: 15),
                        recipeDetails(ingredients),
                      ],
                    ),
                  );
          },
        ),
      ),
    );
  }

  Widget recipeDetails(List<String> ingredients) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('Category: ${recipe['strCategory']}',
            style: const TextStyle(fontSize: 16, fontWeight: FontWeight.w500)),
        const SizedBox(height: 10),
        const Text('Ingredients:',
            style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
        ...ingredients.map((i) => Text('• $i')),
        const SizedBox(height: 15),
        const Text('Instructions:',
            style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
        Text(recipe['strInstructions'] ?? ''),
      ],
    );
  }
}
deployed : https://recipe-finder-flutterr.onrender.com/
