func get_posts(user_id, post_ids):
        if not post_ids:
                return []

        db_posts = f'SELECT id, description, image, created_at FROM post WHERE user_id={user_id} AND id IN {post_ids}'
        db_user = f'SELECT id, username, full_name, profile_picture FROM user WHERE id={user_id}'
        db_user.followed = f'SELECT following_id FROM follow WHERE EXISTS (SELECT following_id FROM follow where following_id={user_id})'
        posts_map = {}

        for post in db_posts:
                setattr(post, 'owner', db_user )
                posts_map[post.id] = post
        
        if posts_map:
                for liked in f'SELECT * FROM like WHERE post_id in {posts_map.keys()}':
                        posts_map[liked.post_id].update({'liked': liked.user_id == user_id})
                
        return [posts_map.get(post_id, None) for post_id in post_ids] if posts_map else []


def merge_two_sorted_posts(first_posts_list, second_posts_list, seen):
        if len(first_posts_list) == 0:
                return second_posts_list[:]
        elif len(second_posts_list) == 0:
                return first_posts_list[:]
        else:
                if first_posts_list[0].get('created_at') == second_posts_list[0].get('created_at'):
                        if first_posts_list[0].get('id') > second_posts_list[0].get('id'):
                                return [first_posts_list[0]] + merge_two_sorted_posts(first_posts_list[1:], second_posts_list, seen)
                        else:
                                return [second_posts_list[0]] + merge_two_sorted_posts(first_posts_list, second_posts_list[1:], seen)

                elif first_posts_list[0].get('created_at') > second_posts_list[0].get('created_at'):
                        return [first_posts_list[0]] + merge_two_sorted_posts(first_posts_list[1:], second_posts_list, seen)
                else:
                        return [second_posts_list[0]] + merge_two_sorted_posts(first_posts_list, second_posts_list[1:], seen)



def merge_posts(list_of_posts):
        list_posts = []
        seen = set()
        for posts in list_of_posts:
                list_posts = merge_two_sorted_posts(list_posts, posts, seen)
                seen.update([post['id'] for post in posts])
        return list_posts
