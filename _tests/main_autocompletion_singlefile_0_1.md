Autocompletion in `main()`
--------------------------

Create the file with the same content as the code section in the "Testing sample" paragraph, open it in the IDE, go to the first line of `main()`, and at the end of every following action make a screenshot, note the results, and comment on them:
- open up autocompletion
- type `std::`
- type `m`
- type `isma`
- remove `misma`, type `imsm`

Tested sample
-------------

```cpp
#include <iostream>
#include <iomanip>
#include <sstream>
#include <fstream>
#include <cassert>
#include <cmath>
#include <vector>
#include <map>
#include <limits>
#include <type_traits>
#include <string>
#include <algorithm>
#include <tuple>
#include <array>
#include <numeric>
#include <memory>
#include <utility>
#include <iterator>
#include <functional>
#include <typeinfo>

template<typename T>
int signum(T a)
{
	if(a < 0) return -1;
	if(a > 0) return 1;
	return 0;
}

#pragma once

class DataType
{
public:
	virtual std::string to_token() const = 0;
	virtual int compare(const DataType&) const = 0;
	virtual ~DataType() {}
};

class String : public DataType
{
	std::string value;
public:
	std::string to_token() const
	{
		return "\"" + value + "\"";
	}
	
	String(const std::string& token = "\"\"")
	{
		value = token.substr(1, token.size()-2);
	}

	int compare(const DataType& rhs_) const
	{
		const auto& rhs = dynamic_cast<const String&>(rhs_);
		return signum(this->value.compare(rhs.value));
	}
};

class Integer : public DataType
{
	int value;

public:
	std::string to_token() const
	{
		std::stringstream ss;
		ss << value;
		return ss.str();
	}

	Integer(const std::string& token = "0")
	{
		std::stringstream ss;
		ss << token;
		ss >> value;
	}

	int compare(const DataType& rhs_) const
	{
		const auto& rhs = dynamic_cast<const Integer&>(rhs_);
		return signum(this->value - rhs.value);
	}
};

class Double : public DataType
{
	double value;

public:
	std::string to_token() const
	{
		std::stringstream ss;
		ss << std::fixed << std::setprecision(3) << value;
		return ss.str();
	}

	Double(const std::string& token = "0.000")
	{
		std::stringstream ss;
		ss << token;
		ss >> value;
	}

	int compare(const DataType& rhs_) const
	{
		const auto& rhs = dynamic_cast<const Double&>(rhs_);
		return signum(this->value - rhs.value);
	}
};

class Boolean : public DataType
{
	bool value;

public:
	std::string to_token() const
	{
		return value ? "prawda" : "falsz";
	}

	Boolean(const std::string& token = "falsz")
	{
		if(token == "prawda") value = true;
		if(token == "falsz") value = false;
	}

	int compare(const DataType& rhs_) const
	{
		const auto& rhs = dynamic_cast<const Boolean&>(rhs_);
		return signum(this->value - rhs.value);
	}
};

template<typename T, T lower_bound, T upper_bound>
class Clamp : public DataType
{
	T value;

public:
	Clamp(T v = lower_bound)
	{
		if(v < lower_bound)
			value = lower_bound;
		else if(v > upper_bound)
			value = upper_bound;
		else
			value = v;
	}

	T getValue() const { return value; }
	operator T() const { return getValue(); }

	int compare(const DataType& rhs_) const
	{
		const auto& rhs = dynamic_cast<const Clamp&>(rhs_);
		return signum(this->getValue() - rhs.getValue());
	}

	std::string to_token() const
	{
		std::stringstream ss;
		ss << getValue();
		return ss.str();
	}

	Clamp(const std::string& token)
	{
		std::stringstream ss;
		ss << token;
		T val;
		ss >> val;
		*this = Clamp(val);
	}
};

template<typename T>
class AutoIncrement : public DataType
{
	T value;
	bool user_defined;
public:
	AutoIncrement(T v) : value(v), user_defined(true) {}
	AutoIncrement() : value(T()), user_defined(false) {}

	void setValue(T v) { user_defined = true; value = v; }
	T getValue() const { return value; }
	operator T() const { return getValue(); }

	int compare(const DataType& rhs_) const
	{
		const auto& rhs = dynamic_cast<const AutoIncrement&>(rhs_);
		return signum(this->value - rhs.value);
	}
	
	std::string to_token() const
	{
		std::stringstream ss;
		ss << getValue();
		return ss.str();
	}
	
	AutoIncrement(const std::string& token)
	{
		if(token.at(0) == '*')
			*this = AutoIncrement();
		else
		{
			std::stringstream ss;
			ss << token;
			T val;
			ss >> val;
			*this = AutoIncrement(val);
		}
	}
};

int get_priority_weight(int p)
{
	if(p == 0)                   //dla priorytetu 0 (użytkownika nie interesuje ten element)
		return 0;                //waga: 0
	if(p < 0)                    //dla priorytetu ujemnego (sortowanie w odwrotnej kolejności)
	{	                         //waga ujemna: -4, -16, 64, ...
		p = -p;                  //dla priorytetu dodatniego (sortowanie normalnie)
		return -(1 << 2*p);      //waga dodatnia: 4, 16, 64, ...
	}                            //specjalnie zostawiam 1 i -1 na specjalne okazje,
	return 1 << 2*p;             //gdyby user nadał priorytet 0 dla wszystkich elementów
	//to wtedy ustawimy priorytet 1 dla ostatniego elementu
	//bo inaczej funkcja porównująca będzie zwracać false dla dowolnych dwóch elementów
}

typedef std::vector<std::string> TokenList;
TokenList leksuj(const std::string& linia);

const int no_autoincrement = -1;

struct Rekord
{
	std::vector<std::shared_ptr<DataType>> dane;
	auto operator[](std::size_t pos) -> decltype(*dane[pos])
	{
		return *dane[pos];
	}
	auto operator[](std::size_t pos) const -> decltype(*dane[pos])
	{
		return *dane[pos];
	}
	void push_back(const std::shared_ptr<DataType>& x)
	{
		return dane.push_back(x);
	}
	template<typename InputIt>
	friend int compare_by_priorities(const Rekord& lhs, const Rekord& rhs, InputIt priority_b, InputIt priority_e)
	{
		assert(lhs.dane.size() == rhs.dane.size());
		int wynik = 0;
		for(std::size_t i = 0; i < lhs.dane.size(); ++i)
		{
			wynik += lhs[i].compare(rhs[i]) * *priority_b++;
		}
		return wynik;
	}
	template<typename InputIt>
	void print(std::ostream& os, InputIt order_b, InputIt order_e, const std::string& separator = " ", bool token = false) const
	{
		std::vector<std::string> rekordy_tokeny;
		for(auto dana_it = dane.begin(); dana_it != dane.end(); ++dana_it)
		{
			rekordy_tokeny.push_back((*dana_it)->to_token());
		}
		const auto order_b_start = order_b;
		for(; order_b != order_e; ++order_b)
		{
			if(order_b_start != order_b)
				os << separator;
			os << rekordy_tokeny[*order_b];
		}
	}
	std::shared_ptr<AutoIncrement<int>> get_autoincrement(int ai)
	{
		if(ai != no_autoincrement)
			return std::dynamic_pointer_cast<AutoIncrement<int>>(dane[ai]);
		else
			return nullptr;
	}
};

struct Table
{
	typedef Rekord value_type;
	typedef std::vector<value_type> container_type;
	typedef container_type::iterator iterator;
	typedef container_type::const_iterator const_iterator;
	container_type rekordy;
	container_type selected_records;
	std::vector<const std::type_info*> column_types;
	std::vector<std::string> column_names;
	std::string table_name;
	int ai;

	std::string name() const
	{
		return table_name;
	}
	std::string column_name_from_position(std::size_t pos) const
	{
		return column_names[pos];
	}
	const std::type_info* column_type_from_position(std::size_t pos) const
	{
		return column_types[pos];
	}
	int position_from_column_name(const std::string& name) const
	{
		auto column_position_it = std::find(column_names.begin(), column_names.end(), name);
		if(column_position_it == column_names.end())
			return -1;
		else
			return std::distance(column_names.begin(), column_position_it);
	}
	std::size_t column_count() const
	{
		return column_types.size();
	}
	int dodaj(value_type v)
	{
		if(rekordy.size() < 1000)
		{
			int next_autoincrement;
			//ustaw pole autoinkrementacji na poprawną wartość, jeżeli tabela ją wspiera
			if(ai != no_autoincrement)
			{
				auto max_element_it = std::max_element(rekordy.begin(), rekordy.end(),
				[this](value_type& lhs, value_type& rhs)
				{
					return *lhs.get_autoincrement(ai) < *rhs.get_autoincrement(ai); 
				});
				if(max_element_it == rekordy.end()) //przedział pusty
					next_autoincrement = 1;
				else //znaleziono element o największej wartości pola autoincrement
					next_autoincrement = *max_element_it->get_autoincrement(ai)+1;
				rekordy.push_back(v);
				*rekordy.back().get_autoincrement(ai) = next_autoincrement;
			}
			else
				rekordy.push_back(v);
			return 1;
		}
		else
			return 0;
	}
	template<typename RanIt, typename ConditionUnaryFunction>
	std::pair<iterator, iterator>
	wybierz(
		RanIt priority_b, 
		RanIt priority_e,
		std::pair<std::size_t, std::size_t> zakres,
		ConditionUnaryFunction condition)
	{
		selected_records.clear();
		std::copy_if(rekordy.begin(), rekordy.end(), std::back_inserter(selected_records), condition);
		std::sort(selected_records.begin(), selected_records.end(), [&](const Rekord& lhs, const Rekord& rhs)
		{
			return compare_by_priorities(lhs, rhs, priority_b, priority_e) < 0;
		});
		return std::make_pair(selected_records.begin() + std::min(zakres.first, selected_records.size()),
			selected_records.begin() + std::min(zakres.first + zakres.second, selected_records.size()));
	}
	template<typename ConditionUnaryFunction>
	int usun(ConditionUnaryFunction condition)
	{
		std::size_t size = rekordy.size();
		rekordy.erase(std::remove_if(rekordy.begin(), rekordy.end(), condition), rekordy.end());
		std::size_t new_size = rekordy.size();
		return size - new_size;
	}
	int usun()
	{
		std::size_t size = rekordy.size();
		rekordy.clear();
		return size;
	}
	void serialize_to(std::ostream& os)
	{
		std::vector<std::size_t> order(column_count());
		std::iota(order.begin(), order.end(), 0);
		for(auto rekord_it = rekordy.begin(); rekord_it != rekordy.end(); ++rekord_it)
		{
			os << "(";
			rekord_it->print(os, order.begin(), order.end(), ", ", true);
			os << ")\n";
		}
	}

	template<typename InputIt1, typename InputIt2>
	Table(std::string name, InputIt1 column_b, InputIt1 column_e, InputIt2 type_b, InputIt2 type_e, int aifield = no_autoincrement) :
		table_name(name),
		column_names(column_b, column_e),
		column_types(type_b, type_e),
		ai(aifield)
	{

	}
};

std::shared_ptr<DataType> make_datatype_from_typeid(const std::type_info* ti, const std::string& s = "")
{
#define REGISTER_NEW_DATATYPE(...) \
do {\
	typedef __VA_ARGS__ type;\
	if(ti == &typeid(type))\
	{\
		if(s == "")\
			return std::make_shared<type>();\
		else\
			return std::make_shared<type>(s);\
	}\
} while(0)
	REGISTER_NEW_DATATYPE(Clamp<int, 10, 600>);
	REGISTER_NEW_DATATYPE(Clamp<int, 1, 10>);
	REGISTER_NEW_DATATYPE(AutoIncrement<int>);
	REGISTER_NEW_DATATYPE(String);
	REGISTER_NEW_DATATYPE(Double);
	REGISTER_NEW_DATATYPE(Boolean);
	REGISTER_NEW_DATATYPE(Integer);
	assert(0);
	return nullptr;
#undef REGISTER_NEW_DATATYPE
}

template<typename RanIt1, typename InputIt2>
Table::value_type make_from_tokens(Table& tablica, RanIt1 tokens_b, RanIt1 tokens_e, InputIt2 order_b, InputIt2 order_e)
{
	Table::value_type retval;
	//nadaj domyślną wartość
	for(std::size_t i = 0; i < tablica.column_count(); ++i)
		retval.dane.push_back(make_datatype_from_typeid(tablica.column_type_from_position(i), ""));
	for(; order_b != order_e; ++order_b)
	{
		retval.dane[*order_b] = make_datatype_from_typeid(
			tablica.column_type_from_position(*order_b),
			*tokens_b++);
	}
	return retval;
}

const std::array<std::string, 1> liczby_init = {
	"wartosc", 
};
const std::array<std::string, 3> studenci_init = {
	"indeks", 
	"imie", 
	"nazwisko", 
};
const std::array<std::string, 3> przedmioty_init = {
	"id", 
	"nazwa", 
	"semestr", 
};
const std::array<std::string, 4> sale_init = {
	"nazwa", 
	"rozmiar", 
	"projektor", 
	"powierzchnia", 
};
const std::array<std::string, 2> wyniki_init = {
	"indeks", 
	"srednia", 
};
const std::array<const std::type_info*, 1> liczby_typy_init = {
	&typeid(Integer),
};
const std::array<const std::type_info*, 3> studenci_typy_init = {
	&typeid(Integer),
	&typeid(String),
	&typeid(String),
};
const std::array<const std::type_info*, 3> przedmioty_typy_init = {
	&typeid(AutoIncrement<int>),
	&typeid(String),
	&typeid(Clamp<int, 1, 10>),
};
const std::array<const std::type_info*, 4> sale_typy_init = {
	&typeid(String),
	&typeid(Clamp<int, 10, 600>),
	&typeid(Boolean),
	&typeid(Double),
};
const std::array<const std::type_info*, 2> wynik_typy_init = {
	&typeid(Integer),
	&typeid(Double),
};

typedef std::pair<std::string, Table> string_table_mapping;
const std::array<string_table_mapping, 5> tablice_init = {
	string_table_mapping("liczby",
		Table("liczby",
			liczby_init.begin(), liczby_init.end(),
			liczby_typy_init.begin(), liczby_typy_init.end())),
	string_table_mapping("studenci",
		Table("studenci",
			studenci_init.begin(), studenci_init.end(),
			studenci_typy_init.begin(), studenci_typy_init.end())),
	string_table_mapping("przedmioty",
		Table("przedmioty",
			przedmioty_init.begin(), przedmioty_init.end(),
			przedmioty_typy_init.begin(), przedmioty_typy_init.end(), 0)),
	string_table_mapping("sale",
		Table("sale",
			sale_init.begin(), sale_init.end(),
			sale_typy_init.begin(), sale_typy_init.end())),
	string_table_mapping("wyniki",
		Table("wyniki",
			wyniki_init.begin(), wyniki_init.end(),
			wynik_typy_init.begin(), wynik_typy_init.end())),
};

typedef std::map<std::string, Table> Tablice;
static Tablice tablice(tablice_init.begin(), tablice_init.end());

Table& get_table(const std::string& name)
{
	return tablice.at(name);
}

void parsuj_strumien(std::istream& is, bool silent = false);
bool parsuj_linie(TokenList::iterator it, TokenList::iterator end, bool silent = false);
void parsuj_dodaj(TokenList::iterator it, TokenList::iterator end, bool silent = false);
void parsuj_wypisz(TokenList::iterator it, TokenList::iterator end, bool silent = false);
void parsuj_usun(TokenList::iterator it, TokenList::iterator end, bool silent = false);
void parsuj_zapisz(TokenList::iterator it, TokenList::iterator end, bool silent = false);
void parsuj_zapiszwszystko(TokenList::iterator it, TokenList::iterator end, bool silent = false);
void parsuj_wczytaj(TokenList::iterator it, TokenList::iterator end, bool silent = false);
void parsuj_wczytajwszystko(TokenList::iterator it, TokenList::iterator end, bool silent = false);
int main()
{
	parsuj_strumien(std::cin);
}
TokenList leksuj(const std::string& linia)
{
	bool is_string = false;
	std::string token;
	TokenList token_list;
	auto flush_token = [&]()
	{
		if(!token.empty())
		{
			token_list.push_back(token);
			token = "";
		}
	};
	for(auto it = linia.begin(); it != linia.end(); ++it)
	{
		if(is_string && *it != '\"')
		{
			token.push_back(*it);
			continue;
		}
		switch(*it)
		{
		case '\"':
			is_string = !is_string;
			token.push_back(*it);
			break;
		case ' ':
		case '\t':
		case ',':
			flush_token();
			break;
		case '(':
		case ')':
		case '!':
		case '<':
		case '>':
		case '=':
			flush_token();
			token_list.push_back(std::string(1, *it));
			break;
		default:
			token.push_back(*it);
		}
	}
	flush_token();
	return token_list;
}
void parsuj_strumien(std::istream& is, bool silent)
{
	std::string line;
	while(std::getline(is, line))
	{
		TokenList result = leksuj(line);
		if(!parsuj_linie(result.begin(), result.end(), silent))
			return;
	}
}
bool parsuj_linie(TokenList::iterator it, TokenList::iterator end, bool silent)
{
	if(it == end) //nic nie wpisano
	{ 
		if(!silent)
			std::cout << "?\n";
	}
	else
	{
		if(*it == "dodaj")
			parsuj_dodaj(std::next(it), end, silent);
		else if(*it == "wypisz")
			parsuj_wypisz(std::next(it), end, silent);
		else if(*it == "usun")
			parsuj_usun(std::next(it), end, silent);
		else if(*it == "zapisz")
			parsuj_zapisz(std::next(it), end, silent);
		else if(*it == "zapiszwszystko")
			parsuj_zapiszwszystko(std::next(it), end, silent);
		else if(*it == "wczytaj")
			parsuj_wczytaj(std::next(it), end, silent);
		else if(*it == "wczytajwszystko")
			parsuj_wczytajwszystko(std::next(it), end, silent);
		else if(*it == "koniec")
			return false;
		else //nieprawidłowa komenda
			if(!silent)
				std::cout << "?\n";
	}
	return true;
}
std::vector<std::size_t> parsuj_kolejnosc_kolumn(TokenList::iterator it, TokenList::iterator end, const Table& tablica, bool silent)
{
	std::vector<std::size_t> order_of_columns;
	auto nazwy_kolumn_it = std::find(it, end, "(");

	if(nazwy_kolumn_it != end)
	{
		for(++nazwy_kolumn_it; *nazwy_kolumn_it != ")"; ++nazwy_kolumn_it)
		{
			auto pozycja = tablica.position_from_column_name(*nazwy_kolumn_it);
			if(pozycja == -1) //nazwa nie jest prawidłową nazwą kolumny
			{
				assert(order_of_columns.empty());
				return order_of_columns;
			}
			order_of_columns.push_back(pozycja);
		}
	}
	return order_of_columns;
}
void parsuj_dodaj_do_tabeli(TokenList::iterator it, TokenList::iterator end, Table& tablica, bool silent)
{
	auto order = parsuj_kolejnosc_kolumn(it, end, tablica, silent);
	auto values_it = it;
	if(!order.empty()) //jest kolejność kolumn
	{
		while(*values_it != ")") //pomiń ją
			 ++values_it;
		++values_it; //pomiń kończący nawias
	}
	else //nie ma kolejności kolumn
	{
		order.resize(tablica.column_count(), 0); //utwórz domyślną kolejność
		std::iota(order.begin(), order.end(), 0);
	}
	++values_it; //pomiń rozpoczynający nawias
	int lrek = tablica.dodaj(make_from_tokens(tablica, values_it, end, order.begin(), order.end()));
	if(!silent)
		std::cout << "Liczba dodanych rekordow: " << lrek << "\n";
}
void parsuj_dodaj(TokenList::iterator it, TokenList::iterator end, bool silent)
{
	parsuj_dodaj_do_tabeli(std::next(it), end, get_table(*it), silent);
}
std::function<bool(const Rekord&)> parsuj_wybor(TokenList::iterator it, TokenList::iterator end, const Table& tablica, bool silent)
{
	auto wybor = std::find(it, end, "gdzie");
	int kryterium_wyboru = -1;
	std::string funkcja_wyboru;
	std::shared_ptr<DataType> wartosc_wyboru;
	if(wybor != end)
	{
		++wybor;
		kryterium_wyboru = tablica.position_from_column_name(*wybor++);
		funkcja_wyboru = *wybor++;
		wartosc_wyboru = make_datatype_from_typeid(tablica.column_type_from_position(kryterium_wyboru), *wybor);
		if(funkcja_wyboru == "<")
			return [=](const Rekord& w){ return w[kryterium_wyboru].compare(*wartosc_wyboru) < 0; };
		if(funkcja_wyboru == ">")
			return [=](const Rekord& w){ return w[kryterium_wyboru].compare(*wartosc_wyboru) > 0; };
		if(funkcja_wyboru == "=")
			return [=](const Rekord& w){ return w[kryterium_wyboru].compare(*wartosc_wyboru) == 0; };
		if(funkcja_wyboru == "!")
			return [=](const Rekord& w){ return w[kryterium_wyboru].compare(*wartosc_wyboru) != 0; };
		assert(0);
		return [](const Rekord&){ return true; };
	}
	else
		return [](const Rekord&){ return true; };
}
void parsuj_wypisz(TokenList::iterator it, TokenList::iterator end, bool silent)
{
	auto& tablica = get_table(*it);

	auto sortowanie = std::find(it, end, "wedlug");
	int kryterium_sortowania = -1;
	int kierunek_sortowania = 1;
	if(sortowanie != end) //mamy podane kryterium sortowania
	{
		++sortowanie;
		if(*sortowanie == "!")
		{
			kierunek_sortowania = -1;
			++sortowanie;
		}
		kryterium_sortowania = tablica.position_from_column_name(*sortowanie);
	}

	auto funkcja_wybierajaca = parsuj_wybor(it, end, tablica, silent);

	auto zakres = std::find(it, end, "zakres");
	std::pair<std::size_t, std::size_t> zakres_wyboru(0, std::numeric_limits<std::size_t>::max());
	if(zakres != end)
	{
		++zakres;
		std::replace(zakres->begin(), zakres->end(), '-', ' ');
		std::stringstream ss(*zakres);
		ss >> zakres_wyboru.first >> zakres_wyboru.second;
	}

	std::array<int, 5> p = { 4, 3, 2, 1 };
	if(kryterium_sortowania != -1)
		p[kryterium_sortowania] = 10 * kierunek_sortowania;
	auto wybrane_rekordy = tablica.wybierz(p.begin(), p.end(), zakres_wyboru, funkcja_wybierajaca);

	std::size_t lrek = std::distance(wybrane_rekordy.first, wybrane_rekordy.second);
	if(!silent)
	{
		std::cout << "Liczba rekordow: " << lrek << "\n";
		if(lrek == 0) return;
		auto order_of_columns = parsuj_kolejnosc_kolumn(it, end, tablica, silent);
		if(order_of_columns.empty())
		{
			order_of_columns.resize(tablica.column_count(), 0);
			std::iota(order_of_columns.begin(), order_of_columns.end(), 0);
		}
		for(auto it = order_of_columns.begin(); it != order_of_columns.end(); ++it)
			std::cout << tablica.column_name_from_position(*it) << " ";
		std::cout << "\n";
		for(auto it = wybrane_rekordy.first; it != wybrane_rekordy.second; ++it)
		{
			it->print(std::cout, order_of_columns.begin(), order_of_columns.end());
			std::cout << "\n";
		}
	}
}
void parsuj_usun(TokenList::iterator it, TokenList::iterator end, bool silent)
{
	auto& tablica = get_table(*it);
	auto warunek = parsuj_wybor(it, end, tablica, silent);
	auto lrek = tablica.usun(warunek);
	if(!silent)
		std::cout << "Liczba usunietych rekordow: " << lrek << "\n";
}
void parsuj_zapisz(TokenList::iterator it, TokenList::iterator end, bool silent)
{
	auto nazwa_tabeli = *it++;
	auto nazwa_pliku = *it++;
	std::ofstream file(nazwa_pliku);
	auto& tablica = get_table(nazwa_tabeli);
	tablica.serialize_to(file);
	if(!silent)
		std::cout << "OK\n";
}
void parsuj_zapiszwszystko(TokenList::iterator it, TokenList::iterator end, bool silent)
{
	for(auto it = tablice.begin(); it != tablice.end(); ++it)
	{
		std::ofstream liczby_file(it->first + ".dat");
		it->second.serialize_to(liczby_file);
	}
	if(!silent)
		std::cout << "OK\n";
}
void parsuj_wczytaj_do_tablicy(std::istream& is, Table& tablica, bool silent)
{
	tablica.usun();
	std::string linia;
	while(std::getline(is, linia))
	{
		TokenList tokeny = leksuj(linia);
		parsuj_dodaj_do_tabeli(tokeny.begin(), tokeny.end(), tablica, true);
	}
}
void parsuj_wczytaj(TokenList::iterator it, TokenList::iterator end, bool silent)
{
	auto nazwa_tabeli = *it++;
	auto nazwa_pliku = *it++;
	std::ifstream file(nazwa_pliku);
	auto& tablica = tablice.at(nazwa_tabeli);
	if(!silent)
	{
		if(!file.good())
			std::cout << "BLAD\n";
		else
			std::cout << "OK\n";
	}
	parsuj_wczytaj_do_tablicy(file, tablica, silent);
}
void parsuj_wczytajwszystko(TokenList::iterator it, TokenList::iterator end, bool silent)
{
	bool error = false;
	for(auto it = tablice.begin(); it != tablice.end(); ++it)
	{
		std::ifstream plik(it->first + ".dat");
		if(!plik.good())
		{
			error = true;
			break;
		}
	}
	for(auto it = tablice.begin(); it != tablice.end(); ++it)
	{
		std::ifstream plik(it->first + ".dat");
		parsuj_wczytaj_do_tablicy(plik, it->second, true);
	}
	if(!silent)
		std::cout << (error ? "BLAD\n" : "OK\n");
}

```